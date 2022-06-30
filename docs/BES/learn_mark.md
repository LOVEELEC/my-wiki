# 程序调用流程

AUDIO_PROMPT_USE_DAC2_ENABLED = 1

IBRT = 1



定义 了

**__AUDIO_QUEUE_SUPPORT__**

**MEDIA_PLAYER_SUPPORT**



**未定义 PROMPT_IN_FLASH**

## 示例 APP_STATUS_INDICATION_POWERON

**app_voice_report(APP_STATUS_INDICATION_POWERON, 0) -->** 

​	**app_voice_report_handler(APP_STATUS_INDICATION_POWERON, 0, true)-->**

​		media_PlayAudio_locally(id, device_id)[POWEROFF调用]	

​        **media_PlayAudio_standalone(AUD_ID_POWER_ON, 0) -->** 



## 函数原型及说明

1. int app_voice_report(APP_STATUS_INDICATION_T status, uint8_t device_id) 
2. int app_voice_report_handler(APP_STATUS_INDICATION_T status, uint8_t device_id, uint8_t isMerging)
3. void media_PlayAudio_locally(AUD_ID_ENUM id, uint8_t device_id)
4. void media_PlayAudio(AUD_ID_ENUM id,uint8_t device_id)
5. void media_PlayAudio_standalone(AUD_ID_ENUM id, uint8_t device_id)





## 提示音文件存储位置

Flash 数据区域，在哪里？如何烧录？如何划分？优缺点？



## app audio

函数的调用：app_init(void)--->app_audio_open();
app_audio_open
（1）初始化提示音模块，调用app_prompt_list_init()完成了内存空间申请，app_prompt_list申请，app_prompt_handler_tid创建；
（2）设置APP_MODUAL_AUDIO模块的回调app_audio_handle_process()，根据当前设置的APP_BT_SETTING_T完成音频流的开启app_bt_stream_open或关闭app_bt_stream_close处理，其内部维护了一个audio队列，依赖于宏定义 __AUDIO_QUEUE_SUPPORT__；

```c
enum APP_MODUAL_ID_T {
    APP_MODUAL_KEY = 0,
    APP_MODUAL_AUDIO,
    APP_MODUAL_BATTERY,
    APP_MODUAL_BT,
    APP_MODUAL_FM,
    APP_MODUAL_SD,
    APP_MODUAL_LINEIN,
    APP_MODUAL_USBHOST,
    APP_MODUAL_USBDEVICE,
    APP_MODUAL_WATCHDOG,
    APP_MODUAL_AUDIO_MANAGE,
    APP_MODUAL_ANC,
    APP_MODUAL_VOICE_ASSIST,
    APP_MODUAL_SMART_MIC,
#ifdef __PC_CMD_UART__  
    APP_MODUAL_CMD,
#endif
#ifdef TILE_DATAPATH
    APP_MODUAL_TILE,
#endif
    APP_MODUAL_MIC,
#ifdef VOICE_DETECTOR_EN
    APP_MODUAL_VOICE_DETECTOR,
#endif
#ifdef AUDIO_HEARING_COMPSATN
    APP_MODUAL_HEAR_COMP,
#endif
    APP_MODUAL_OHTER,

    APP_MODUAL_NUM
};

app_set_threadhandle(APP_MODUAL_AUDIO, app_audio_handle_process);

enum APP_BT_SETTING_T {
    APP_BT_SETTING_OPEN = 0,
    APP_BT_SETTING_CLOSE,
    APP_BT_SETTING_SETUP,
    APP_BT_SETTING_RESTART,
    APP_BT_SETTING_CLOSEALL,
    APP_BT_SETTING_CLOSEMEDIA,
    APP_BT_SETTING_NUM,
};

typedef struct {
    uint32_t message_id;
    uint32_t message_ptr;
    uint32_t message_Param0;
    uint32_t message_Param1;
    uint32_t message_Param2;
} APP_MESSAGE_BODY;

static int app_audio_handle_process(APP_MESSAGE_BODY *msg_body)

```

（3）bt流触发器初始化app_bt_stream_init()，创建定时器，在音乐或电话流开启时启动定时器回调app_bt_stream_trigger_timeout_cb()向模块APP_MODUAL_AUDIO发消息进入回调app_audio_handle_process完成相应操作

```c
osTimerCreate(osTimer(APP_BT_STREAM_TRIGGER_TIMEOUT), osTimerOnce, NULL);
app_audio_sendrequest_param(APP_BT_STREAM_A2DP_SBC, (uint8_t)APP_BT_SETTING_RESTART, 0, 0)
app_audio_sendrequest(APP_BT_STREAM_HFP_PCM, (uint8_t)APP_BT_SETTING_RESTART, 0);
```



## App_bt_media_manager模块

app_audio_manager_open

函数内流程：

（1）设置APP_MODUAL_AUDIO_MANAGE模块回调app_audio_manager_handle_process()，根据当前BT_MEDIA_MANAGER完成音乐、电话、提示音的开启bt_media_start或关闭bt_media_stop处理。

```c
enum APP_BT_MEDIA_MANAGER_ID_T {
    APP_BT_STREAM_MANAGER_START = 0,
    APP_BT_STREAM_MANAGER_STOP,
    APP_BT_STREAM_MANAGER_STOP_MEDIA,
    APP_BT_STREAM_MANAGER_UPDATE_MEDIA,
    APP_BT_STREAM_MANAGER_SWAP_SCO,
    APP_BT_STREAM_MANAGER_CTRL_VOLUME,
    APP_BT_STREAM_MANAGER_TUNE_SAMPLERATE_RATIO,
    APP_BT_STREAM_MANAGER_NUM,
};

static int app_audio_manager_handle_process(APP_MESSAGE_BODY *msg_body)
    
```

（2） 通过调用audio_prompt_init_handler()---> memset((uint8_t *)&audio_prompt_env, 0, sizeof(audio_prompt_env));将提示音AUDIO_PROMPT_ENV_T初始化为0；

（3） APP_BT_SETTING_T的产生过程分音乐、电话、提示音三部分介绍



## audioflinger(音频管理器)

1. 主要功能？
2. 程序框架
3. 

梳理笔记记录：
af_open 
​	. 初始化 af_stream_cfg_t结构体
    . 创建thread(af_thread),,并利用信号量机制完成线程同步；

```c
//pingpong machine
enum AF_PP_T{
    PP_PING = 0,
    PP_PANG = 1
};

enum AUD_STREAM_USE_DEVICE_T{
    AUD_STREAM_USE_DEVICE_NULL = 0,
    AUD_STREAM_USE_EXT_CODEC,
    AUD_STREAM_USE_I2S0_MASTER,
    AUD_STREAM_USE_I2S0_SLAVE,
    AUD_STREAM_USE_I2S1_MASTER,
    AUD_STREAM_USE_I2S1_SLAVE,
    AUD_STREAM_USE_TDM0_MASTER,
    AUD_STREAM_USE_TDM0_SLAVE,
    AUD_STREAM_USE_TDM1_MASTER,
    AUD_STREAM_USE_TDM1_SLAVE,
    AUD_STREAM_USE_INT_CODEC,
    AUD_STREAM_USE_INT_CODEC2,
    AUD_STREAM_USE_INT_SPDIF,
    AUD_STREAM_USE_BT_PCM,
    AUD_STREAM_USE_DPD_RX,
    AUD_STREAM_USE_MC,
};

struct af_stream_ctl_t{
    enum AF_PP_T pp_index;      //pingpong operate
    uint8_t pp_cnt;             //use to count the lost signals
    uint8_t status;             //status machine
    enum AUD_STREAM_USE_DEVICE_T use_device;

    uint32_t hdlr_intvl_ticks;
    uint32_t prev_hdlr_time;
    bool first_hdlr_proc;
    bool intvl_check_en;
};

struct AF_STREAM_CONFIG_T {
    enum AUD_SAMPRATE_T sample_rate;        // 采样率
    enum AUD_CHANNEL_MAP_T channel_map;     // 音频通道map
    enum AUD_CHANNEL_NUM_T channel_num;     // 通道编号
    enum AUD_BITS_T bits;                   // 采样宽度(8\12\16\20\24\32)
    enum AUD_STREAM_USE_DEVICE_T device;    // 音频使用设备(i2S\DMA等)
    enum AUD_IO_PATH_T io_path;             // 音频输入来源及输出方式(SPEAKER)
    enum AUD_DATA_ALIGN_T align;            // 数据对齐方式
    enum AUD_FS_FIRST_EDGE_T fs_edge;       // 采样方式
    uint16_t fs_cycles;
    uint8_t slot_cycles;
    bool chan_sep_buf;
    bool sync_start;
    //should define type
    uint8_t vol;

    AF_STREAM_HANDLER_T handler;            

    uint8_t *data_ptr;
    uint32_t data_size;
};

struct af_stream_cfg_t {
    //used inside
    struct af_stream_ctl_t ctl;

    //dma buf parameters, RAM can be alloced in different way
    uint8_t *dma_buf_ptr;
    uint32_t dma_buf_size;

    //store stream cfg parameters
    struct AF_STREAM_CONFIG_T cfg;

    //dma cfg parameters
#ifdef DYNAMIC_AUDIO_BUFFER_COUNT
    uint8_t dma_desc_cnt;
#endif
    struct HAL_DMA_DESC_T dma_desc[MAX_AUDIO_BUFFER_COUNT];
    struct HAL_DMA_CH_CFG_T dma_cfg;

    //callback function
    AF_STREAM_HANDLER_T handler;
};
```



线程中根据不同的信号量，调用af_thread_stream_handler对不同类型的数据流进行处理

static struct af_stream_cfg_t af_stream\[AUD_STREAM_ID_NUM]\[AUD_STREAM_NUM];
AUD_STREAM_ID_NUM = 4 AUD_STREAM_NUM =2
[AUD_STREAM_ID_x(0,3)]\[AUD_STREAM_PLAYBACK=0]
[AUD_STREAM_ID_x(0,3)]\[AUD_STREAM_CAPTURE=1]

这个具体怎么对应？





## 提示音

**app_voice_report**

函数内流程：

（1） app_voice_report(APP_STATUS_INDICATION_T status, uint8_t device_id)是提示音总入口函数，APP_STATUS_INDICATION_T 枚举类型参数表示提示音序号，用来索引提示音ID  ，device_id  表示设备号，一般是0.

```c
typedef enum APP_STATUS_INDICATION_T {
    APP_STATUS_INDICATION_POWERON = 0,
    APP_STATUS_INDICATION_INITIAL,
    APP_STATUS_INDICATION_PAGESCAN,
    ...
```

（2）根据相关宏定义选择不同的提示音播放方式

```c
void media_PlayAudio_locally(AUD_ID_ENUM id, uint8_t device_id)//仅本地播放
void media_PlayAudio_standalone_locally(AUD_ID_ENUM id, uint8_t device_id)//打断播放
void media_PlayAudio(AUD_ID_ENUM id,uint8_t device_id)//混合播放
```

（3） 如果是从耳发送命令给对端主耳播放app_tws_let_peer_device_play_audio_prompt()----->tws_ctrl_send_cmd()，如果是主耳发送提示音任务请求app_prompt_push_request()---->app_prompt_list_append(app_prompt_list, &req);---->osSignalSet(app_prompt_handler_tid, PROMPT_HANDLER_SIGNAL_NEW_PROMPT_REQ);

**app_prompt_handler_thread**

函数的调用：app_prompt_list_init()--->app_prompt_handler_thread()

函数内流程： 

（1）收到信号PROMPT_HANDLER_SIGNAL_NEW_PROMPT_REQ后,根据当前是否有正在播放的提示音以及app_prompt_list是否为空，通过调用app_prompt_refresh_list来实现相关功能。

 （2）收到信号PPROMPT_HANDLER_SIGNAL_CLEAR_REQ后，先清除app_prompt_list，然后混合模式下发送PROMPT_HANDLER_SIGNAL_PLAYING_COMPLETED信号停止提示音，最后通过app_audio_sendrequest(APP_PLAY_BACK_AUDIO, (uint8_t)APP_BT_SETTING_CLOSE, devId)---->app_audio_handle_process()完成提示音流的关闭。

（3）收到信号PROMPT_HANDLER_SIGNAL_PLAYING_COMPLETED，先标识当前没有正在播放的提示音，然后停止提示音保护定时器app_prompt_protector_timer，最后，调用app_prompt_refresh_list来实现相关功能。
