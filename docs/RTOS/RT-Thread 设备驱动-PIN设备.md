## RT-Thread 设备驱动-PIN设备

虽然各厂商芯片有关GPIO的功能及SDK会有所不同，但无外乎也就这几样功能。

RT-Thread这套pin设备框架为我们提供了良好的通用接口，降低我们换芯片后的学习成本。按照这些函数指针的类型编写相应的驱动函数，就可以实现对接。

上层会通过访问这些函数指针来间接地调用底层的驱动函数，是不是有点像虚函数表。

应用程序通过 RT-Thread 提供的 PIN 设备管理接口来访问 GPIO，相关接口如下所示：

| **函数**            | **描述**             |
| ------------------- | -------------------- |
| rt_pin_get()        | 获取引脚编号         |
| rt_pin_mode()       | 设置引脚模式         |
| rt_pin_write()      | 设置引脚电平         |
| rt_pin_read()       | 读取引脚电平         |
| rt_pin_attach_irq() | 绑定引脚中断回调函数 |
| rt_pin_irq_enable() | 使能引脚中断         |
| rt_pin_detach_irq() | 脱离引脚中断回调函数 |

```c
struct rt_pin_ops
{
    void (*pin_mode)(struct rt_device *device, rt_base_t pin, rt_uint8_t mode);
    void (*pin_write)(struct rt_device *device, rt_base_t pin, rt_uint8_t value);
    rt_ssize_t  (*pin_read)(struct rt_device *device, rt_base_t pin);
    rt_err_t (*pin_attach_irq)(struct rt_device *device, rt_base_t pin,
            rt_uint8_t mode, void (*hdr)(void *args), void *args);
    rt_err_t (*pin_detach_irq)(struct rt_device *device, rt_base_t pin);
    rt_err_t (*pin_irq_enable)(struct rt_device *device, rt_base_t pin, rt_uint8_t enabled);
    rt_base_t (*pin_get)(const char *name);
#ifdef RT_USING_DM
    rt_err_t (*pin_irq_mode)(struct rt_device *device, rt_base_t pin, rt_uint8_t mode);
    rt_ssize_t (*pin_parse)(struct rt_device *device, struct rt_ofw_cell_args *args, rt_uint32_t *flags);
#endif
#ifdef RT_USING_PINCTRL
    rt_err_t (*pin_ctrl_confs_apply)(struct rt_device *device, void *fw_conf_np);
#endif /* RT_USING_PINCTRL */
};


const static struct rt_pin_ops gd32_pin_ops =
{
    .pin_mode = gd32_pin_mode,
    .pin_write = gd32_pin_write,
    .pin_read = gd32_pin_read,
    .pin_attach_irq = gd32_pin_attach_irq,
    .pin_detach_irq= gd32_pin_detach_irq,
    .pin_irq_enable = gd32_pin_irq_enable,
    RT_NULL,
};
```



```c

/**
  * @brief  set pin mode
  * @param  dev, pin, mode
  * @retval None
  */
static void gd32_pin_mode(rt_device_t dev, rt_base_t pin, rt_uint8_t mode)
{
    const struct pin_index *index = RT_NULL;
    rt_uint32_t pin_mode = 0;

#if defined SOC_SERIES_GD32F4xx
      rt_uint32_t pin_pupd = 0, pin_odpp = 0;
#endif

    index = get_pin(pin);
    if (index == RT_NULL)
    {
        return;
    }

    /* GPIO Periph clock enable */
    rcu_periph_clock_enable(index->clk);
#if defined SOC_SERIES_GD32F4xx
        pin_mode = GPIO_MODE_OUTPUT;
#else
    pin_mode = GPIO_MODE_OUT_PP;
#endif

    switch(mode)
    {
    case PIN_MODE_OUTPUT:
        /* output setting */
#if defined SOC_SERIES_GD32F4xx
        pin_mode = GPIO_MODE_OUTPUT;
        pin_pupd = GPIO_PUPD_NONE;
        pin_odpp = GPIO_OTYPE_PP;
#else
        pin_mode = GPIO_MODE_OUT_PP;
#endif
        break;
    case PIN_MODE_OUTPUT_OD:
        /* output setting: od. */
#if defined SOC_SERIES_GD32F4xx
        pin_mode = GPIO_MODE_OUTPUT;
        pin_pupd = GPIO_PUPD_NONE;
        pin_odpp = GPIO_OTYPE_OD;
#else
        pin_mode = GPIO_MODE_OUT_OD;
#endif
        break;
    case PIN_MODE_INPUT:
        /* input setting: not pull. */
#if defined SOC_SERIES_GD32F4xx
        pin_mode = GPIO_MODE_INPUT;
        pin_pupd = GPIO_PUPD_PULLUP | GPIO_PUPD_PULLDOWN;
#else
        pin_mode = GPIO_MODE_IN_FLOATING;
#endif
        break;
    case PIN_MODE_INPUT_PULLUP:
        /* input setting: pull up. */
#if defined SOC_SERIES_GD32F4xx
        pin_mode = GPIO_MODE_INPUT;
        pin_pupd = GPIO_PUPD_PULLUP;
#else
        pin_mode = GPIO_MODE_IPU;
#endif
        break;
    case PIN_MODE_INPUT_PULLDOWN:
        /* input setting: pull down. */
#if defined SOC_SERIES_GD32F4xx
        pin_mode = GPIO_MODE_INPUT;
        pin_pupd = GPIO_PUPD_PULLDOWN;
#else
        pin_mode = GPIO_MODE_IPD;
#endif
        break;
    default:
            break;
    }

#if defined SOC_SERIES_GD32F4xx
    gpio_mode_set(index->gpio_periph, pin_mode, pin_pupd, index->pin);
    if(pin_mode == GPIO_MODE_OUTPUT)
    {
        gpio_output_options_set(index->gpio_periph, pin_odpp, GPIO_OSPEED_50MHZ, index->pin);
    }
#else
        gpio_init(index->gpio_periph, pin_mode, GPIO_OSPEED_50MHZ, index->pin);
#endif
}

/**
  * @brief  pin write
  * @param  dev, pin, valuie
  * @retval None
  */
static void gd32_pin_write(rt_device_t dev, rt_base_t pin, rt_uint8_t value)
{
    const struct pin_index *index = RT_NULL;

    index = get_pin(pin);
    if (index == RT_NULL)
    {
        return;
    }

    gpio_bit_write(index->gpio_periph, index->pin, (bit_status)value);
}

/**
  * @brief  pin read
  * @param  dev, pin
  * @retval None
  */
static rt_ssize_t gd32_pin_read(rt_device_t dev, rt_base_t pin)
{
    rt_ssize_t value = PIN_LOW;
    const struct pin_index *index = RT_NULL;

    index = get_pin(pin);
    if (index == RT_NULL)
    {
        return -RT_EINVAL;
    }

    value = gpio_input_bit_get(index->gpio_periph, index->pin);
    return value;
}
```

```c
rt_inline rt_int32_t bit2bitno(rt_uint32_t bit)
{
    rt_uint8_t i;
    for (i = 0; i < 32; i++)
    {
        if ((0x01 << i) == bit)
        {
            return i;
        }
    }
    return -1;
}

/**
  * @brief  pin write
  * @param  pinbit
  * @retval None
  */
rt_inline const struct pin_irq_map *get_pin_irq_map(rt_uint32_t pinbit)
{
    rt_int32_t map_index = bit2bitno(pinbit);
    if (map_index < 0 || map_index >= ITEM_NUM(pin_irq_map))
    {
        return RT_NULL;
    }
    return &pin_irq_map[map_index];
}

/**
  * @brief  pin irq attach
  * @param  device, pin, mode
  * @retval None
  */
static rt_err_t gd32_pin_attach_irq(struct rt_device *device, rt_base_t pin,
                              rt_uint8_t mode, void (*hdr)(void *args), void *args)
{
    const struct pin_index *index = RT_NULL;
    rt_base_t level;
    rt_int32_t hdr_index = -1;

    index = get_pin(pin);
    if (index == RT_NULL)
    {
        return -RT_EINVAL;
    }

    hdr_index = bit2bitno(index->pin);
    if (hdr_index < 0 || hdr_index >= ITEM_NUM(pin_irq_map))
    {
        return -RT_EINVAL;
    }

    level = rt_hw_interrupt_disable();
    if (pin_irq_hdr_tab[hdr_index].pin == pin &&
        pin_irq_hdr_tab[hdr_index].hdr == hdr &&
        pin_irq_hdr_tab[hdr_index].mode == mode &&
        pin_irq_hdr_tab[hdr_index].args == args)
    {
        rt_hw_interrupt_enable(level);
        return RT_EOK;
    }
    if (pin_irq_hdr_tab[hdr_index].pin != -1)
    {
        rt_hw_interrupt_enable(level);
        return -RT_EFULL;
    }
    pin_irq_hdr_tab[hdr_index].pin = pin;
    pin_irq_hdr_tab[hdr_index].hdr = hdr;
    pin_irq_hdr_tab[hdr_index].mode = mode;
    pin_irq_hdr_tab[hdr_index].args = args;
    rt_hw_interrupt_enable(level);

    return RT_EOK;
}

/**
  * @brief  pin irq detach
  * @param  device, pin
  * @retval None
  */
static rt_err_t gd32_pin_detach_irq(struct rt_device *device, rt_base_t pin)
{
    const struct pin_index *index = RT_NULL;
    rt_base_t level;
    rt_int32_t hdr_index = -1;

    index = get_pin(pin);
    if (index == RT_NULL)
    {
        return -RT_EINVAL;
    }

    hdr_index = bit2bitno(index->pin);
    if (hdr_index < 0 || hdr_index >= ITEM_NUM(pin_irq_map))
    {
        return -RT_EINVAL;
    }

    level = rt_hw_interrupt_disable();
    if (pin_irq_hdr_tab[hdr_index].pin == -1)
    {
        rt_hw_interrupt_enable(level);
        return RT_EOK;
    }
    pin_irq_hdr_tab[hdr_index].pin = -1;
    pin_irq_hdr_tab[hdr_index].hdr = RT_NULL;
    pin_irq_hdr_tab[hdr_index].mode = 0;
    pin_irq_hdr_tab[hdr_index].args = RT_NULL;
    rt_hw_interrupt_enable(level);

    return RT_EOK;
}

/**
  * @brief  pin irq enable
  * @param  device, pin, enabled
  * @retval None
  */
static rt_err_t gd32_pin_irq_enable(struct rt_device *device, rt_base_t pin, rt_uint8_t enabled)
{
    const struct pin_index *index;
    const struct pin_irq_map *irqmap;
    rt_base_t level;
    rt_int32_t hdr_index = -1;
    exti_trig_type_enum trigger_mode;

    index = get_pin(pin);
    if (index == RT_NULL)
    {
        return -RT_EINVAL;
    }

    if (enabled == PIN_IRQ_ENABLE)
    {
        hdr_index = bit2bitno(index->pin);
        if (hdr_index < 0 || hdr_index >= ITEM_NUM(pin_irq_map))
        {
            return -RT_EINVAL;
        }

        level = rt_hw_interrupt_disable();
        if (pin_irq_hdr_tab[hdr_index].pin == -1)
        {
            rt_hw_interrupt_enable(level);
            return -RT_EINVAL;
        }

        irqmap = &pin_irq_map[hdr_index];

        switch (pin_irq_hdr_tab[hdr_index].mode)
        {
            case PIN_IRQ_MODE_RISING:
                trigger_mode = EXTI_TRIG_RISING;
                break;
            case PIN_IRQ_MODE_FALLING:
                trigger_mode = EXTI_TRIG_FALLING;
                break;
            case PIN_IRQ_MODE_RISING_FALLING:
                trigger_mode = EXTI_TRIG_BOTH;
                break;
            default:
                rt_hw_interrupt_enable(level);
                return -RT_EINVAL;
        }

#if defined SOC_SERIES_GD32F4xx
        rcu_periph_clock_enable(RCU_SYSCFG);
#else
        rcu_periph_clock_enable(RCU_AF);
#endif

        /* enable and set interrupt priority */
        nvic_irq_enable(irqmap->irqno, 5U, 0U);

        /* connect EXTI line to  GPIO pin */
#if defined SOC_SERIES_GD32F4xx
        syscfg_exti_line_config(index->port_src, index->pin_src);
#else
        gpio_exti_source_select(index->port_src, index->pin_src);
#endif

        /* configure EXTI line */
        exti_init((exti_line_enum)(index->pin), EXTI_INTERRUPT, trigger_mode);
        exti_interrupt_flag_clear((exti_line_enum)(index->pin));

        rt_hw_interrupt_enable(level);
    }
    else if (enabled == PIN_IRQ_DISABLE)
    {
        irqmap = get_pin_irq_map(index->pin);
        if (irqmap == RT_NULL)
        {
            return -RT_EINVAL;
        }
        nvic_irq_disable(irqmap->irqno);
    }
    else
    {
        return -RT_EINVAL;
    }

    return RT_EOK;
}
```

```c
#define PIN_IRQ_MODE_RISING 0x00         /* 上升沿触发 */
#define PIN_IRQ_MODE_FALLING 0x01        /* 下降沿触发 */
#define PIN_IRQ_MODE_RISING_FALLING 0x02 /* 边沿触发（上升沿和下降沿都触发）*/
#define PIN_IRQ_MODE_HIGH_LEVEL 0x03     /* 高电平触发 */
#define PIN_IRQ_MODE_LOW_LEVEL 0x04      /* 低电平触发 */
```



### [获取引脚编号](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pin/pin?id=获取引脚编号)

RT-Thread 提供的引脚编号需要和芯片的引脚号区分开来，它们并不是同一个概念，引脚编号由 PIN 设备驱动程序定义，和具体的芯片相关。有3种方式可以获取引脚编号： API 接口获取、使用宏定义或者查看PIN 驱动文件。

#### [使用 API](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pin/pin?id=使用-api)

使用 rt_pin_get() 获取引脚编号，如下获取 PF9 的引脚编号：

```c
pin_number = rt_pin_get("PF.9");复制错误复制成功
```

#### [使用宏定义](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pin/pin?id=使用宏定义)

如果使用 `rt-thread/bsp/gd32` 目录下的 BSP 则可以使用下面的宏获取引脚编号：

```c
GET_PIN(port, pin)复制错误复制成功
```

获取引脚号为 PF9 的 LED0 对应的引脚编号的示例代码如下所示：

```c
#include <rtdevice.h>
#include <board.h>    //如果不包含，可能会遇到错误提示没有'F'定义

#define LED0_PIN        GET_PIN(F,  9)复制错误复制成功
```

#### [查看驱动文件](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pin/pin?id=查看驱动文件)

如果使用其他 BSP 则需要查看 PIN 驱动代码 drv_gpio.c 文件确认引脚编号。此文件里有一个数组存放了每个 PIN 脚对应的编号信息，如下所示：

```c
static const struct pin_index pins[] =
{
#ifdef GPIOA
    GD32_PIN(0,  A, 0),
    GD32_PIN(1,  A, 1),
    GD32_PIN(2,  A, 2),
    GD32_PIN(3,  A, 3),
    GD32_PIN(4,  A, 4),
    GD32_PIN(5,  A, 5),
    GD32_PIN(6,  A, 6),
    GD32_PIN(7,  A, 7),
    GD32_PIN(8,  A, 8),
    GD32_PIN(9,  A, 9),
    GD32_PIN(10, A, 10),
    GD32_PIN(11, A, 11),
    GD32_PIN(12, A, 12),
    GD32_PIN(13, A, 13),
    GD32_PIN(14, A, 14),
    GD32_PIN(15, A, 15),
#endif
#ifdef GPIOB
    GD32_PIN(16, B, 0),
    GD32_PIN(17, B, 1),
    GD32_PIN(18, B, 2),
    GD32_PIN(19, B, 3),
    GD32_PIN(20, B, 4),
    GD32_PIN(21, B, 5),
    GD32_PIN(22, B, 6),
    GD32_PIN(23, B, 7),
    GD32_PIN(24, B, 8),
    GD32_PIN(25, B, 9),
    GD32_PIN(26, B, 10),
    GD32_PIN(27, B, 11),
    GD32_PIN(28, B, 12),
    GD32_PIN(29, B, 13),
    GD32_PIN(30, B, 14),
    GD32_PIN(31, B, 15),
#endif
#ifdef GPIOC
    GD32_PIN(32, C, 0),
    GD32_PIN(33, C, 1),
    GD32_PIN(34, C, 2),
    GD32_PIN(35, C, 3),
    GD32_PIN(36, C, 4),
    GD32_PIN(37, C, 5),
    GD32_PIN(38, C, 6),
    GD32_PIN(39, C, 7),
    GD32_PIN(40, C, 8),
    GD32_PIN(41, C, 9),
    GD32_PIN(42, C, 10),
    GD32_PIN(43, C, 11),
    GD32_PIN(44, C, 12),
    GD32_PIN(45, C, 13),
    GD32_PIN(46, C, 14),
    GD32_PIN(47, C, 15),
#endif
#ifdef GPIOD
    GD32_PIN(48, D, 0),
    GD32_PIN(49, D, 1),
    GD32_PIN(50, D, 2),
    GD32_PIN(51, D, 3),
    GD32_PIN(52, D, 4),
    GD32_PIN(53, D, 5),
    GD32_PIN(54, D, 6),
    GD32_PIN(55, D, 7),
    GD32_PIN(56, D, 8),
    GD32_PIN(57, D, 9),
    GD32_PIN(58, D, 10),
    GD32_PIN(59, D, 11),
    GD32_PIN(60, D, 12),
    GD32_PIN(61, D, 13),
    GD32_PIN(62, D, 14),
    GD32_PIN(63, D, 15),
#endif
#ifdef GPIOE
    GD32_PIN(64, E, 0),
    GD32_PIN(65, E, 1),
    GD32_PIN(66, E, 2),
    GD32_PIN(67, E, 3),
    GD32_PIN(68, E, 4),
    GD32_PIN(69, E, 5),
    GD32_PIN(70, E, 6),
    GD32_PIN(71, E, 7),
    GD32_PIN(72, E, 8),
    GD32_PIN(73, E, 9),
    GD32_PIN(74, E, 10),
    GD32_PIN(75, E, 11),
    GD32_PIN(76, E, 12),
    GD32_PIN(77, E, 13),
    GD32_PIN(78, E, 14),
    GD32_PIN(79, E, 15),
#endif
#ifdef GPIOF
    GD32_PIN(80, F, 0),
    GD32_PIN(81, F, 1),
    GD32_PIN(82, F, 2),
    GD32_PIN(83, F, 3),
    GD32_PIN(84, F, 4),
    GD32_PIN(85, F, 5),
    GD32_PIN(86, F, 6),
    GD32_PIN(87, F, 7),
    GD32_PIN(88, F, 8),
    GD32_PIN(89, F, 9),
    GD32_PIN(90, F, 10),
    GD32_PIN(91, F, 11),
    GD32_PIN(92, F, 12),
    GD32_PIN(93, F, 13),
    GD32_PIN(94, F, 14),
    GD32_PIN(95, F, 15),
#endif
#ifdef GPIOG
    GD32_PIN(96, G, 0),
    GD32_PIN(97, G, 1),
    GD32_PIN(98, G, 2),
    GD32_PIN(99, G, 3),
    GD32_PIN(100, G, 4),
    GD32_PIN(101, G, 5),
    GD32_PIN(102, G, 6),
    GD32_PIN(103, G, 7),
    GD32_PIN(104, G, 8),
    GD32_PIN(105, G, 9),
    GD32_PIN(106, G, 10),
    GD32_PIN(107, G, 11),
    GD32_PIN(108, G, 12),
    GD32_PIN(109, G, 13),
    GD32_PIN(110, G, 14),
    GD32_PIN(111, G, 15),
#endif
#ifdef GPIOH
    GD32_PIN(112, H, 0),
    GD32_PIN(113, H, 1),
    GD32_PIN(114, H, 2),
    GD32_PIN(115, H, 3),
    GD32_PIN(116, H, 4),
    GD32_PIN(117, H, 5),
    GD32_PIN(118, H, 6),
    GD32_PIN(119, H, 7),
    GD32_PIN(120, H, 8),
    GD32_PIN(121, H, 9),
    GD32_PIN(122, H, 10),
    GD32_PIN(123, H, 11),
    GD32_PIN(124, H, 12),
    GD32_PIN(125, H, 13),
    GD32_PIN(126, H, 14),
    GD32_PIN(127, H, 15),
#endif
#ifdef GPIOI
    GD32_PIN(128, I, 0),
    GD32_PIN(129, I, 1),
    GD32_PIN(130, I, 2),
    GD32_PIN(131, I, 3),
    GD32_PIN(132, I, 4),
    GD32_PIN(133, I, 5),
    GD32_PIN(134, I, 6),
    GD32_PIN(135, I, 7),
    GD32_PIN(136, I, 8),
    GD32_PIN(137, I, 9),
    GD32_PIN(138, I, 10),
    GD32_PIN(139, I, 11),
    GD32_PIN(140, I, 12),
    GD32_PIN(141, I, 13),
    GD32_PIN(142, I, 14),
    GD32_PIN(143, I, 15),
#endif
};
```

以`GD32_PIN(2, A, 15)`为例，2 为 RT-Thread 使用的引脚编号，A 为端口号，15 为引脚号，所以 PA15 对应的引脚编号为 2。

### [设置引脚模式](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pin/pin?id=设置引脚模式)

引脚在使用前需要先设置好输入或者输出模式，通过如下函数完成：

```c
void rt_pin_mode(rt_base_t pin, rt_base_t mode);复制错误复制成功
```

| **参数** | **描述**     |
| -------- | ------------ |
| pin      | 引脚编号     |
| mode     | 引脚工作模式 |

目前 RT-Thread 支持的引脚工作模式可取如所示的 5 种宏定义值之一，每种模式对应的芯片实际支持的模式需参考 PIN 设备驱动程序的具体实现：

```c
#define PIN_MODE_OUTPUT 0x00            /* 输出 */
#define PIN_MODE_INPUT 0x01             /* 输入 */
#define PIN_MODE_INPUT_PULLUP 0x02      /* 上拉输入 */
#define PIN_MODE_INPUT_PULLDOWN 0x03    /* 下拉输入 */
#define PIN_MODE_OUTPUT_OD 0x04         /* 开漏输出 */复制错误复制成功
```

使用示例如下所示：

```c
#define BEEP_PIN_NUM            35  /* PB0 */

/* 蜂鸣器引脚为输出模式 */
rt_pin_mode(BEEP_PIN_NUM, PIN_MODE_OUTPUT);复制错误复制成功
```

### [设置引脚电平](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pin/pin?id=设置引脚电平)

设置引脚输出电平的函数如下所示：

```c
void rt_pin_write(rt_base_t pin, rt_base_t value);复制错误复制成功
```

| **参数** | **描述**                                                     |
| -------- | ------------------------------------------------------------ |
| pin      | 引脚编号                                                     |
| value    | 电平逻辑值，可取 2 种宏定义值之一：PIN_LOW 低电平，PIN_HIGH 高电平 |

使用示例如下所示：

```c
#define BEEP_PIN_NUM            35  /* PB0 */

/* 蜂鸣器引脚为输出模式 */
rt_pin_mode(BEEP_PIN_NUM, PIN_MODE_OUTPUT);
/* 设置低电平 */
rt_pin_write(BEEP_PIN_NUM, PIN_LOW);复制错误复制成功
```

### [读取引脚电平](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pin/pin?id=读取引脚电平)

读取引脚电平的函数如下所示：

```c
int rt_pin_read(rt_base_t pin);复制错误复制成功
```

| **参数** | **描述** |
| -------- | -------- |
| pin      | 引脚编号 |
| **返回** | ——       |
| PIN_LOW  | 低电平   |
| PIN_HIGH | 高电平   |

使用示例如下所示：

```c
#define BEEP_PIN_NUM            35  /* PB0 */
int status;

/* 蜂鸣器引脚为输出模式 */
rt_pin_mode(BEEP_PIN_NUM, PIN_MODE_OUTPUT);
/* 设置低电平 */
rt_pin_write(BEEP_PIN_NUM, PIN_LOW);

status = rt_pin_read(BEEP_PIN_NUM);复制错误复制成功
```

### [绑定引脚中断回调函数](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pin/pin?id=绑定引脚中断回调函数)

若要使用到引脚的中断功能，可以使用如下函数将某个引脚配置为某种中断触发模式并绑定一个中断回调函数到对应引脚，当引脚中断发生时，就会执行回调函数:

```c
rt_err_t rt_pin_attach_irq(rt_int32_t pin, rt_uint32_t mode,
                           void (*hdr)(void *args), void *args);复制错误复制成功
```

| **参数** | **描述**                                   |
| -------- | ------------------------------------------ |
| pin      | 引脚编号                                   |
| mode     | 中断触发模式                               |
| hdr      | 中断回调函数，用户需要自行定义这个函数     |
| args     | 中断回调函数的参数，不需要时设置为 RT_NULL |
| **返回** | ——                                         |
| RT_EOK   | 绑定成功                                   |
| 错误码   | 绑定失败                                   |

中断触发模式 mode 可取如下 5 种宏定义值之一：

```c
#define PIN_IRQ_MODE_RISING 0x00         /* 上升沿触发 */
#define PIN_IRQ_MODE_FALLING 0x01        /* 下降沿触发 */
#define PIN_IRQ_MODE_RISING_FALLING 0x02 /* 边沿触发（上升沿和下降沿都触发）*/
#define PIN_IRQ_MODE_HIGH_LEVEL 0x03     /* 高电平触发 */
#define PIN_IRQ_MODE_LOW_LEVEL 0x04      /* 低电平触发 */复制错误复制成功
```

使用示例如下所示：

```c
#define KEY0_PIN_NUM            55  /* PD8 */
/* 中断回调函数 */
void beep_on(void *args)
{
    rt_kprintf("turn on beep!\n");

    rt_pin_write(BEEP_PIN_NUM, PIN_HIGH);
}
static void pin_beep_sample(void)
{
    /* 按键0引脚为输入模式 */
    rt_pin_mode(KEY0_PIN_NUM, PIN_MODE_INPUT_PULLUP);
    /* 绑定中断，下降沿模式，回调函数名为beep_on */
    rt_pin_attach_irq(KEY0_PIN_NUM, PIN_IRQ_MODE_FALLING, beep_on, RT_NULL);
}复制错误复制成功
```

### [使能引脚中断](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pin/pin?id=使能引脚中断)

绑定好引脚中断回调函数后使用下面的函数使能引脚中断：

```c
rt_err_t rt_pin_irq_enable(rt_base_t pin, rt_uint32_t enabled);复制错误复制成功
```

| **参数** | **描述**                                                     |
| -------- | ------------------------------------------------------------ |
| pin      | 引脚编号                                                     |
| enabled  | 状态，可取 2 种值之一：PIN_IRQ_ENABLE（开启），PIN_IRQ_DISABLE（关闭） |
| **返回** | ——                                                           |
| RT_EOK   | 使能成功                                                     |
| 错误码   | 使能失败                                                     |

使用示例如下所示：

```c
#define KEY0_PIN_NUM            55  /* PD8 */
/* 中断回调函数 */
void beep_on(void *args)
{
    rt_kprintf("turn on beep!\n");

    rt_pin_write(BEEP_PIN_NUM, PIN_HIGH);
}
static void pin_beep_sample(void)
{
    /* 按键0引脚为输入模式 */
    rt_pin_mode(KEY0_PIN_NUM, PIN_MODE_INPUT_PULLUP);
    /* 绑定中断，下降沿模式，回调函数名为beep_on */
    rt_pin_attach_irq(KEY0_PIN_NUM, PIN_IRQ_MODE_FALLING, beep_on, RT_NULL);
    /* 使能中断 */
    rt_pin_irq_enable(KEY0_PIN_NUM, PIN_IRQ_ENABLE);
}复制错误复制成功
```

### [脱离引脚中断回调函数](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pin/pin?id=脱离引脚中断回调函数)

可以使用如下函数脱离引脚中断回调函数：

```c
rt_err_t rt_pin_detach_irq(rt_int32_t pin);复制错误复制成功
```

| **参数** | **描述** |
| -------- | -------- |
| pin      | 引脚编号 |
| **返回** | ——       |
| RT_EOK   | 脱离成功 |
| 错误码   | 脱离失败 |

引脚脱离了中断回调函数以后，中断并没有关闭，还可以调用绑定中断回调函数再次绑定其他回调函数。

```c
#define KEY0_PIN_NUM            55  /* PD8 */
/* 中断回调函数 */
void beep_on(void *args)
{
    rt_kprintf("turn on beep!\n");

    rt_pin_write(BEEP_PIN_NUM, PIN_HIGH);
}
static void pin_beep_sample(void)
{
    /* 按键0引脚为输入模式 */
    rt_pin_mode(KEY0_PIN_NUM, PIN_MODE_INPUT_PULLUP);
    /* 绑定中断，下降沿模式，回调函数名为beep_on */
    rt_pin_attach_irq(KEY0_PIN_NUM, PIN_IRQ_MODE_FALLING, beep_on, RT_NULL);
    /* 使能中断 */
    rt_pin_irq_enable(KEY0_PIN_NUM, PIN_IRQ_ENABLE);
    /* 脱离中断回调函数 */
    rt_pin_detach_irq(KEY0_PIN_NUM);
}复制错误复制成功
```

## [PIN 设备使用示例](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pin/pin?id=pin-设备使用示例)

PIN 设备的具体使用方式可以参考如下示例代码，示例代码的主要步骤如下：

1. 设置蜂鸣器对应引脚为输出模式，并给一个默认的低电平状态。
2. 设置按键 0 和 按键1 对应引脚为输入模式，然后绑定中断回调函数并使能中断。
3. 按下按键 0 蜂鸣器开始响，按下按键 1 蜂鸣器停止响。

```c
/*
 * 程序清单：这是一个 PIN 设备使用例程
 * 例程导出了 pin_beep_sample 命令到控制终端
 * 命令调用格式：pin_beep_sample
 * 程序功能：通过按键控制蜂鸣器对应引脚的电平状态控制蜂鸣器
*/

#include <rtthread.h>
#include <rtdevice.h>

/* 引脚编号，通过查看设备驱动文件drv_gpio.c确定 */
#ifndef BEEP_PIN_NUM
    #define BEEP_PIN_NUM            35  /* PB0 */
#endif
#ifndef KEY0_PIN_NUM
    #define KEY0_PIN_NUM            55  /* PD8 */
#endif
#ifndef KEY1_PIN_NUM
    #define KEY1_PIN_NUM            56  /* PD9 */
#endif

void beep_on(void *args)
{
    rt_kprintf("turn on beep!\n");

    rt_pin_write(BEEP_PIN_NUM, PIN_HIGH);
}

void beep_off(void *args)
{
    rt_kprintf("turn off beep!\n");

    rt_pin_write(BEEP_PIN_NUM, PIN_LOW);
}

static void pin_beep_sample(void)
{
    /* 蜂鸣器引脚为输出模式 */
    rt_pin_mode(BEEP_PIN_NUM, PIN_MODE_OUTPUT);
    /* 默认低电平 */
    rt_pin_write(BEEP_PIN_NUM, PIN_LOW);

    /* 按键0引脚为输入模式 */
    rt_pin_mode(KEY0_PIN_NUM, PIN_MODE_INPUT_PULLUP);
    /* 绑定中断，下降沿模式，回调函数名为beep_on */
    rt_pin_attach_irq(KEY0_PIN_NUM, PIN_IRQ_MODE_FALLING, beep_on, RT_NULL);
    /* 使能中断 */
    rt_pin_irq_enable(KEY0_PIN_NUM, PIN_IRQ_ENABLE);

    /* 按键1引脚为输入模式 */
    rt_pin_mode(KEY1_PIN_NUM, PIN_MODE_INPUT_PULLUP);
    /* 绑定中断，下降沿模式，回调函数名为beep_off */
    rt_pin_attach_irq(KEY1_PIN_NUM, PIN_IRQ_MODE_FALLING, beep_off, RT_NULL);
    /* 使能中断 */
    rt_pin_irq_enable(KEY1_PIN_NUM, PIN_IRQ_ENABLE);
}
/* 导出到 msh 命令列表中 */
MSH_CMD_EXPORT(pin_beep_sample, pin beep sample);
```