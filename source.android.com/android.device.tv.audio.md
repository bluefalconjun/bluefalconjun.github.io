
###[**TV Audio**](http://source.android.com/devices/audio/tv.html)

-----
在TV音频的实现中, **TV Input Framework(TIF)**管理器使用audio路由相关的API来支持复杂的多路音频输出的改变. 当设备上的SoC实现TV的HAL时, 每一个TV Input(HDMI In, Tuner等)都需要提供**`TVInputHardwareInfo`**结构, 其中必须说明**`AudioPort`**信息, 来标明支持的音频类型/地址的信息.

 - 物理存在的音频输入/输出设备拥有一个相关的**`AudioPort`**. 

 - 软件实现的音频输出/输入流由**`AudioMixPort`**来代表(它是**`AudioPort`**的子类).

在以上的基础上, TIF使用AudioPort相关的信息来选择使用音频路由处理的API.

![TIF audio](http://source.android.com/devices/audio/images/ape_audio_tv_tif.png)
**`Figure 1. TV Input Framework (TIF)`电视输入框架**

-----
> **Requirements**

-----
设备商必须要实现支持以下音频路由选择的音频HAL:

**`Audio Ports`**	:

 - TV音频输入必须包含相关的音频源(sourfce)端口实现.
 - TV音频输出必须包含相关的音频端(sink)端口实现.
 - 任何Tv Input/Tv Output的音频端口间必须都能够创建音频通路(audio patch).

**`Default Input`**:

 - **`AudioRecord`**(默认创建的输入源)必须是Android TV通过**`AUIDIO_DEVICE_IN_DEFAULT`**来获取的默认虚拟的空输入源.
**`(默认使用该空源, 以保证在设备不提供相关Port时/实现相关Port时不会出现空端口的情况.)`**

**`Device Loopback:`**	

 - 需要实现支持**`AUDIO_DEVICE_IN_LOOPBACK`**类型的输入设备, 用来完整的混合所有当前TV输出的音频输出信号(包括11Khz,16位单声道/48Khz,16位单声道)到该处输入. 它仅用来进行音频采集.

-----
> **TV audio devices**

-----
android对于TV音频输入/输出支持以下类型的音频设备:
参见**`system/media/audio/include/system/audio.h`**

**`Note: 在Android 5.1 或早期版本,该文件位于: system/core/include/system/audio.h`**

    /* output devices */
    AUDIO_DEVICE_OUT_AUX_DIGITAL  = 0x400,
    AUDIO_DEVICE_OUT_HDMI   = AUDIO_DEVICE_OUT_AUX_DIGITAL,
    /* HDMI Audio Return Channel */
    AUDIO_DEVICE_OUT_HDMI_ARC   = 0x40000,
    /* S/PDIF out */
    AUDIO_DEVICE_OUT_SPDIF    = 0x80000,

    /* input devices */
    AUDIO_DEVICE_IN_AUX_DIGITAL   = AUDIO_DEVICE_BIT_IN | 0x20,
    AUDIO_DEVICE_IN_HDMI      = AUDIO_DEVICE_IN_AUX_DIGITAL,
    /* TV tuner input */
    AUDIO_DEVICE_IN_TV_TUNER    = AUDIO_DEVICE_BIT_IN | 0x4000,
    /* S/PDIF in */
    AUDIO_DEVICE_IN_SPDIF   = AUDIO_DEVICE_BIT_IN | 0x10000,
    AUDIO_DEVICE_IN_LOOPBACK    = AUDIO_DEVICE_BIT_IN | 0x40000,

**`设备商一般可以自行扩展该设备定义`**

-----
> **Audio HAL extension**

-----
Audio HAL扩展中关于audio路由API部分的定义在以下文件中:
**`system/media/audio/include/system/audio.h`**

**`Note: 在Android 5.1和早期版本中, 该文件位于:system/core/include/system/audio.h`**

    /* audio port configuration structure used to specify a particular configuration of an audio port */
    struct audio_port_config {
        audio_port_handle_t      id;           /* port unique ID */
        audio_port_role_t        role;         /* sink or source */
        audio_port_type_t        type;         /* device, mix ... */
        unsigned int             config_mask;  /* e.g AUDIO_PORT_CONFIG_ALL */
        unsigned int             sample_rate;  /* sampling rate in Hz */
        audio_channel_mask_t     channel_mask; /* channel mask if applicable */
        audio_format_t           format;       /* format if applicable */
        struct audio_gain_config gain;         /* gain to apply if applicable */
        union {
            struct audio_port_config_device_ext  device;  /* device specific info */
            struct audio_port_config_mix_ext     mix;     /* mix specific info */
            struct audio_port_config_session_ext session; /* session specific info */
        } ext;
    };
    struct audio_port {
        audio_port_handle_t      id;                /* port unique ID */
        audio_port_role_t        role;              /* sink or source */
        audio_port_type_t        type;              /* device, mix ... */
        unsigned int             num_sample_rates;  /* number of sampling rates in following array */
        unsigned int             sample_rates[AUDIO_PORT_MAX_SAMPLING_RATES];
        unsigned int             num_channel_masks; /* number of channel masks in following array */
        audio_channel_mask_t     channel_masks[AUDIO_PORT_MAX_CHANNEL_MASKS];
        unsigned int             num_formats;       /* number of formats in following array */
        audio_format_t           formats[AUDIO_PORT_MAX_FORMATS];
        unsigned int             num_gains;         /* number of gains in following array */
        struct audio_gain        gains[AUDIO_PORT_MAX_GAINS];
        struct audio_port_config active_config;     /* current audio port configuration */
        union {
            struct audio_port_device_ext  device;
            struct audio_port_mix_ext     mix;
            struct audio_port_session_ext session;
        } ext;
    };

**`每个audio port可以使用多个audio port config中的一个. 勇士按照当前port类型, 按照类型进行扩展.`**

**`hardware/libhardware/include/hardware/audio.h`**

    struct audio_hw_device {
      :
        /**
         * Routing control
         */
        /* Creates an audio patch between several source and sink ports.
         * The handle is allocated by the HAL and should be unique for this
         * audio HAL module. */
        int (*create_audio_patch)(struct audio_hw_device *dev,
                                   unsigned int num_sources,
                                   const struct audio_port_config *sources,
                                   unsigned int num_sinks,
                                   const struct audio_port_config *sinks,
                                   audio_patch_handle_t *handle);
        /* Release an audio patch */
        int (*release_audio_patch)(struct audio_hw_device *dev,
                                   audio_patch_handle_t handle);
        /* Fills the list of supported attributes for a given audio port.
         * As input, "port" contains the information (type, role, address etc...)
         * needed by the HAL to identify the port.
         * As output, "port" contains possible attributes (sampling rates, formats,
         * channel masks, gain controllers...) for this port.
         */
        int (*get_audio_port)(struct audio_hw_device *dev,
                              struct audio_port *port);
        /* Set audio port configuration */
        int (*set_audio_port_config)(struct audio_hw_device *dev,
                             const struct audio_port_config *config);

**`对于创建audio patch来说, audio device 和  audio_patch_handle是对应绑定的, 在该device中可以指定多个sources/sink`**


-----
> **Testing DEVICE_IN_LOOPBACK**

-----
在TV开发过程中测试**`DEVICE_IN_LOOPBACK`**, 可以使用以下代码, 当测试完成后, 所捕获的音频保存到/sdcard/record_loopback.raw, 可以使用ffmpeg进行监听.

    测试代码:略
    监听代码:略


-----
> **Use cases**

-----
**TV tuner with speaker output**

当TV tuner被激活时, audio路由API将为tuner和默认的输出(如:扬声器)建立audio patch. tuner的输出并需要进行解码, 但是最终的音频输出将会同软件的输出流进行混合播放(电视播放时,使用按键进行操作同时输出按键音).

![tv tuner to speaker](http://source.android.com/devices/audio/images/ape_audio_tv_tuner.png)
**`Figure 2. TV tuner同扬声器输出的Audio Patch.`**

**HDMI OUT during live TV**

用户在观看直播电视时, 将输出切换至HDMI音频输出(此时系统接收到**`Intent.ACTION_HDMI_AUDIO_PLUG`广播**). 此时所有的流的输出设备将切换至**`HDMI_OUT`**端口, 由TIF管理器将已存在的tuner audio patch的sink端口切换至HDMI_OUT端口.

![hdmi out](http://source.android.com/devices/audio/images/ape_audio_tv_hdmi_tuner.png)
**`Figure 3. 直播电视在HDMI_OUT时的Audio Patch.`**

-----