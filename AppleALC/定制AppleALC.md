<b>1、提取codec</b></br>
通过CLOVER提取codec</br>
操作方法：</br>
•    CLOVER v2.4k r4886或者以上</br>
•    CLOVER里要有AudioDex-64.efi</br>
•    CLOVER引导界面，按F8快捷键，它会在EFI/CLOVER/misc目录下生成以HdaCodec开头的文件，其中的HdaCodec#x (Realtek ALCxxx).txt就是你需要提取的codec</br>
通过linux提取codec</br>
操作方法：任意一个linux或口袋LINUX启动后，进入路劲proc/asound/card1，将文件夹里面的所有名为codec#开头的文件都复制出来，其中一个就是你的声卡codec，千万不要把HDMI音频的文件复制出来了（如果card1没有就一定在card0文件夹，笔记本一般都在card1中）</br>

<b>2、找出有效的节点,通过WINDOWS HDU TOOLS获取更容易</b></br>

Speaker(内置喇叭)                0x14   0x2--0xc--0x14      转为10进制  20>12>2 输出设备从后向前顺序 </br> 
Headphone(外置耳机)            0x15   0x3--0xd--0x15     转为10进制  21>13>3  输出设备从后向前顺序</br>
MIC(外插麦克）                    0x18   0x8--0x23--0x18   转为10进制   8>35>24  输入设备从前向后顺序</br>
MIC(内置麦克)                      0x12   0x9--0x22--0x12    转为10进制  9>34>18   输入设备从前向后顺序</br></br>

这是这个机器有效的四个节点，节点ID为0x14,0x15,0x18,0x12</br>
windows下HDU TOOLS记录下的信息</br>
0x14  misc:0x1  color:unknow   connection type:Other analog  DefaultDevice:Speaker</br>
Directional Location:NA   Location:Internal Port Connectivity:Fixed Function Device
0x15   misc:0x1  color:black   connection type:1/8 stero/mono  DefaultDevice:HP Out</br>
Directional Location:left   Location:External Port Connectivity:Jack
0x12   misc:0x1  color:unknow  connection type:Other Digital  DefaultDevice:Mic in</br>
Directional Location:N/A  Location:External Port Connectivity:Fixed Function Device
0x18  misc:0x1  color:black  connection type:1/8 stero/mono  DefaultDevice:Mic in</br>
Directional Location:Left  Location:External Port Connectivity:Jack</br>

<b>3、整理ConfigData,通过Pin Configurtor软件进行修改获取</b></br>
有些输出设备具有EAPD，软件会自动显示出来，简单判断EAPD节点的方法:那就是它通常会位于Speaker Out和HP Out这两个输出节点上.至于其它教程提到过的关于01470c02是组神奇的代码,可以让外放发声的说法是错误的,它可能刚好声卡的Speaker Out的输出节点是0x14而已.如果您的Speaker Out输出节点是0x16,那么就需要把它修改为01670c02,当然要遵守这个公式:Address+节点+71c+02</br>
插口位置</br>
1/8" stereo/mono 通常接口</br>
Optical SPDIF光纤口 </br> 
Rear（后）笔记本选这个，意义不大</br>
Front（前）</br>
ATAPI （內建）笔记本一般选这个</br>
•    Fixed 内部设备</br>
•    Jack  是通过插孔进行连接的外部设备</br>
•    N/A   是其它未知设备</br>
•    内置麦克风 —— Mic In</br>
•    内置扬声器 ——Speaker</br>
•    线路输出 —— line out</br>
•    外置麦克风 —— Mic at Ext</br>
•    线路输入 —— Line In 笔记本外置麦克要选这个</br>
•    耳机 —— HP Out</br>
•    
•    获取到的pinconfig数据</br>
01471C10 01471D01 01471E17 01471F99 01571D10 01571E21 01571F01 01570C02 01871C30 01871D18 01871E81 01871F01 01271C40 01271D01 01271EA6 01271F99 01470C02</br>
   
<b>4、修改AppleALC源码配置文件</b></br>
•    最后确认一共需要修改和定制的为4个文件：</br>
•    1.Applealc——resources——alc269—>Platforms28.xml.zlib</br>
•    2.Applealc——resources——alc269—>layout28.xml.zlib</br>
•    3.Applealc—>resources—>alc269—>info.Plist</br>
•    4.Applealc—>resources—>pinconfigs.kext—>contents—>info.Plist</br>
 
外置mic需要修改电压控制值来实现外置mic驱动。</br>
搜索codec中外置mic下的vref值，vref含义为初始电压基础上增加的百分比，如图为vref为50。当vref不为Hiz时，muteGPIO={（vref转换为16进制）+"0100"+node id}转换为10进制，codec中vref表示的是十进制，计算时转为16进制。如：在节点 0x18发现vref_80，80转换为16进制=50，则muteGPIO=（50010018）转换为10进制=838926360；如果vref为Hiz，则muteGPIO=0</br>

<b>5、编译AppleALC为AppleALC.KEXT</b></br>
打开源码里的xcode项目进行编译，生成驱动文件，注意别忘记了要放lilu.kext在源码根目录，要不然会出错的~</br>
<b>6、放入驱动，注入layout-id即可</b></br>

教程相关软件下载：https://github.com/btwise/TOOLS/AppleALC
<b>视频教程观看地址：https://www.bilibili.com/video/av50809670/</b>
