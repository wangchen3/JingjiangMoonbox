#define DWVERSION 2126111913
//2126111913|| 开关自动governor（KS_CONF_AUTO_SW=1，ks_conf_err.h，宏=0回退人机手动主权）：bit13新飞点/bit15置信度接入按"速度≥50m/min且纵向误差首次|err|<20丝（0.2mm，aiError_V[0]显示平均口径，丢失哨兵不算）→自动打开；速度<48m/min→自动关闭并清收敛锁存（须重新达标才再开）；48~50迟滞带保持原状（防边界抖动致飞点反复复位暖机）"自动控制。人机手动置1=强制打开（抵自动关）、置0=解除强制交还自动（手动检测=镜像位≠governor上拍回写值）；生效状态变化时ARM回写镜像位，屏显始终=实际生效状态；bit14置信度预估/显示上电默认置1常开（仍可手动关显示）。实现：modbus485.c的KsConfAutoSwTick，main主循环每拍无条件调用（不依赖主站通讯/精采样状态，机器停转也能及时关）；状态机20场景仿真全过。屏侧开关控件语义变化：从"手动开关"变为"实际状态显示+强制开入口"（置1强制开，置0跟随自动）|Yang.zj+开关自动governor 2026.11.19.133703

#define DWVERSION 2126111911
//2126111911|| Qn噪声窗排除区半宽 W25→0.5×W25（ks_conf_err.c：lwE=f25-0.5×W25、rwS=b25+0.5×W25，基线区[0,f25-0.5W25)+[b25+0.5W25,门尾)）：原排除区3×W25在窄门/宽标工况（0x03实测W25=8.36mm@20mm门居中）把两侧空白全吃光→噪声窗饿死Qn无效显0（τ调大救不了空窗）；半标宽后排除区=2×W25，脚部覆盖仍足够，σ略抬升属可接受代价。老架构ARM与mark_quality.py同批同步；qt仿真平台opt链待同步（KSOptCalcQn窗口同式）；②Qn容差τ默认 5→12丝（现场实测5偏小Qn压分过严：KS_CONF_TAU_SILK=12.0f，上电镜像wConfTau自动跟随显示12，人机可调范围仍1~100，与排除区半标宽同批）|Yang.zj+排除区半标宽/τ默认12 2026.11.19.112423|114221

#define DWVERSION 2126111817
//2126111817|| 置信度IIR修正量前馈直通（记忆平移）：E_k=o_iVResult是"N幅平均后已减修正量"的口径，修正量阶跃被IIR平滑成慢爬坡（客户实测修正量修正后机器反应变慢）；现Process内每圈检测ΔA=本圈修正量-上圈修正量，变化时同步平移混合记忆s_iPrevOutV/H（数学恒等于"先IIR滤波、最后再减修正量"），修正量变化当圈瞬时穿透输出，IIR只平滑物理测量；丢失圈期间的变化恢复首圈生效；只改ks_conf_err.c，开关关闭时行为不变|Yang.zj+修正量记忆平移 2026.11.18.173201
//（1720为协作方提号，无独立日志，内容含在1716/1817批次内）

#define DWVERSION 2126111716
//2126111716|| ①偏移自调整目标基线128→64（KS_FB_DacOutCtl.c的KS_OffsetAutoCtl_T1T2调用参数，8bit粗采样中点改为64）；②置信度Qn容差τ变量化：原编译期固定KS_CONF_TAU_SILK=5丝改为全局rConfTauSilk（默认5），人机485下发wConfTau（读写3180，单位丝1:1直通——寄存器5=算法5丝，人机控件勿带×10（s50控件有×10但τ没有）；有效1~100，bit14=1才生效，越界保持原值，下一圈生效）；③CK/Qn子分数观测上报：rConfCk/QnLastV/H每圈记录（0~100，有效圈刷新丢失圈保持），打包wConfCk/wConfQn@读写3181/3182（高8位纵向|低8位横向，四舍五入钳位0~100，与wConfPack同款；bit14=0时送0）；④上电镜像初始化（KSModbusHmiComm入口一次性）：wConfS50/wConfTau通讯镜像置为模块默认80/5——否则通讯建立时按区重发会把BSS零值镜像整区写入人机，屏上预设的τ初值5被刷成0（算法侧因0不在1~100范围仍用默认5，纯显示不一致；修复后屏显80/5与算法一致，开关字仍0不初始化）|Yang.zj+上电参数镜像初始化 2026.11.17.200800

#define DWVERSION 2126111315
//2126111315|| 人机485运行期开关接入（自老架构移植回）：data_RW.wConfSwitch bit13飞点/bit14置信度观察/bit15接入（读写43177，默认全关=上电原行为）、wConfS50参数下发（读写43178，0.001mm，1~1000）、置信度显示data_RW.wConfPack（读写3179，高8纵向F/低8横向F；2026.11.16提交0e80898由只写131挪入，dynaData_WO_Ex还原全预留）；新旧飞点路径常驻编译运行期选择，开关沿翻转自动复位剔除器。通道明细见README/置信度飞点_人机485通道说明.md，整体移植总结见README/飞点剔除与双置信度_移植总结.md|Yang.zj+移植总结文档 2026.11.13|通道地址随0e80898更新 2026.11.16

#define DWVERSION 2126111312
//2126111312|| markDetection(全自动套印色标检测)封编译期宏USE_MARK_DETECTION：暂未投用默认0——markDetection.c整体#if包裹不编译(零RAM/Flash，代码完整保留未删除)，main.c的include与KsAutoFindSignal调用点一并剔除；启用改宏为1重编译即可|Yang.zj+markDetection封编译宏 2026.11.13.120400

#define DWVERSION 2126111311
//2126111311|| 新增双置信度融合误差：自老架构ARM(Slave_stm32_ARM 2026.11.12版)整体移植ks_conf_err.c/h，去运行期开关化改为编译期总开关KS_USE_CONF_ERR(ks_conf_err.h，1=融合误差直接替换上报iVerror/iHerror，0=完全回退原行为重新编译)；算法逐行保留(CK跨圈一致性+Qn测量随机精度，F=√(C×Qn/100)，out_k=F·E_k+(1-F)·out_{k-1})，详见双置信度与飞点剔除_汇报文档.md|Yang.zj+置信度迁移8.4 2026.11.13.112036
;	# KESAI\Process\ks_conf_err.c / ks_conf_err.h（新增文件）
;	# 每圈置信度计算：KS_FB_ProSample.c中KS_CalcError_ARC之后调用KS_ConfErr_Process
;	# 误差链路接入：上报处iVerror/iHerror用KS_ConfErr_GetV/H()替换（#if KS_USE_CONF_ERR）
;	# 老架构的485运行期开关(46059 bit3/bit4)与6206显示不迁移；RAM约+13.2KB

#define DWVERSION 2022041301
//2022041301|| ???ó′?2é?ù?????ú1é??ó2?2022.4.13.151211

;	# KESAI\Process\calcerr.c
;	# 新增两个函数
;	# INT KS_GetSignalDetails()// 获取信号特征细节
;	# INT KS_CalcErrUseRSdata()// 通过粗采样数据计算套印误差/门内当前色和前一色/以及当前色和第1色的误差/找寻第1色的原则是当前色根据顺逆序20mm*（色序-1），适用于常规色标/不考虑跨色及插标等方式
;	# 调用接口增加到
;	# 
;	# USER\main.c增加
;	# static WORD g_wCnt_Rs = 0;
;	# static INT g_iRtn_Calc2022 = 0;
;	# static INT g_aiVResult[2] = {0};
;	# 
;	# 
;	# GT_RN_SampleRoundUpdata()前增加如下代码：
;	# ///////////////////////////////////////////
;	# 			g_wCnt_Rs++;
;	# 			// 2022.4.13.145245
;	# 			REAL rGirth = 620;
;	# 			if( stStAxisParam.lrGirth!=0 ) {
;	# 				rGirth = stStAxisParam.lrGirth;
;	# 			}
;	# 			g_iRtn_Calc2022 =
;	# 			KS_CalcErrUseRSdata(
;	# 				pbyRoundData1,//BYTE* i_pbyT1Rs,
;	# 				pbyRoundData2,//BYTE* i_pbyT2Rs,
;	# 				alrBase[0],//REAL i_rBaselineT1Rs,
;	# 				alrBase[1],//REAL i_rBaselineT2Rs,
;	# 				stStAxisParam.iGateSa,//INT i_iGateADDR,
;	# 				stStAxisParam.iGateWidthRondPt,//INT i_iGateWidthPointNR_Rs,
;	# 				(INT)(20.0*4000/rGirth),//INT i_i20mmWidthPointNR_Rs,
;	# 				stStAxisParam.iMarkTypeId,//INT i_iMarkTypId,
;	# 				stStAxisParam.iMarkArrangeModal,//INT i_iArrangeTypId,
;	# 				stStAxisParam.iAxisId,//INT i_iColorId,
;	# 				rGirth,// 2022.4.13.183007/repair bug.
;	# 				g_aiVResult);//,//REAL o_arVResult[2],
;	# 				//g_arHResult);//REAL o_arHResult[2])
;	# ///////////////////////////////////////////
;	# 通讯内容利用奥特马上报5#命令中的预热温度及状态等内容，复用上报main的全局定义信息。


//	char *sVersion="KESAI_GTSD_II_Slave version:2020.5.15.160306";//版本号：
	/*
		1.增加屏保延时和屏保状态切换
		2.保存文件相关
		3.横向快速对版
	*/
//	char *sVersion="KESAI_GTSD_II_Slave version:2020.05.12.163203";//版本号：其余版本记录详细见readme.txt
	/*
		1.增加看门狗相关
	*/	
	
//	char *sVersion="KESAI_GTSD_II_Slave version:2020.05.08.091428";//版本号：
	/*
		1.存放历史误差数组深度改为50
		2.接收成功标志和100ms标志关联
	*/
//	char *sVersion="KESAI_GTSD_II_Slave version:2020.05.05.172422";//版本号：
	/*
		1.改进接收中断处理
		2.威纶人机宏指令更新
		3.发送周期更改为6个周期-读-发-读-发-读-发
	*/
//	char *sVersion="KESAI_GTSD_II_Slave version:2020.04.24.113838";//版本号：
	/*
		优化计算基线效率问题前-算基线采用两个循环-后续放在一个循环里
	*/
//	char *sVersion="KESAI_GTSD_II_Slave version:2020.04.21.094043";//版本号：
	/*
		整理程序，删除多余变量，检查逻辑重复
		i_pxInclDeputyStation 
		i_piDeputyStationId
		副主站通讯相关
	*/	
//	char *sVersion="KESAI_GTSD_II_Slave version:2020.4.18.171857";//版本号：
	/*
		将变换频率较低参数，改变再发送
	*/	
//	char *sVersion="KESAI_GTSD_II_Slave version:2020.04.01.115921";//版本号：
	/*
		将计算误差修改为单组波门
	*/	

//	char *sVersion="KESAI_GTSD_II_Slave version:2020.03.31.200322";//版本号：
	/*
		合并远程升级代码，准备将计算误差修改为单组波门
	*/	


//	char *sVersion="KESAI_GTSD_II_Slave version:2020.03.26.182543";//版本号：
	/*2020.03.26.182547
		485发送内容分开-单向发送及双向通讯内容
	*/	


//	char *sVersion="KESAI_GTSD_II_Slave version:2020.03.24.154304";//版本号：
	/*2020.03.24.154304
		该版本前使用查询发送-更新为中断发送，中断接收
	*/	

//	char *sVersion="KESAI_GTSD_II_Slave version:2020.03.20.164514";//版本号：
	/*
		2020.03.20.164532|波门双向通讯正常d
	*/

//	char *sVersion="KESAI_GTSD_II_Slave version:2020.03.19.174325";//版本号：
	/*
		2020.03.19.174339：首次联机调试程序：目前问题：1.485双向通讯
		2.上行参数 3.首次运行程序不知在哪等待 4.
	*/
//	char *sVersion="KESAI_GTSD_II_Slave version:2020.03.12.133950";//版本号：
	/*
		2020.03.12.133950：编译通过，测试版-程序正常运行，无死循环
	*/
//	char *sVersion="KESAI_GTSD_II_Slave version:2020.03.06.165741";//版本号：
	/*
		2020.03.06.170641：初始版本-合并黄工源代码
	*/
