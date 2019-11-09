# rsfMRI 云端处理平台编写简介

0. DICOM数据 转为nii格式

建议调用MRIcroGL中的dcm2niix.exe 程序来完成，其可以更精确的将DICOM数据转换为nii/nii.gz格式，并输出.json 文件导出DICOM头文件信息；而后去除前5/10张fMRI图像减小磁场不稳定干扰

预处理：

课题组常规使用spm进行预处理，其预处理过程主要有slice-timing-》 realign-》将T1像配准到realign后的功能像的平均上-》分割T1像得到配准T1到MNI空间的参数-》将realign后的功能像配准到MNI空间上-》detrend-》filter （0.01-0.1 Hz）-》regressout nuisance signals （headmotion (6个参数)，fMRI在WM里的均值，fMRI在CSF里的均值，fMRI全脑信息均值（可选），及以上8/9个值的diff））
1.	Detrend：

去除线性成分，可参考程序y = spm_detrend(x,p)

输入x为每个voxel的fMRI time-series

p为阶数，一般取1，即去除线性成分

输出y为去除线性成分后的time-series

2.	Filter：

可参考 [Fil_data] = gretna_filtering(Data, SamplePeriod, FreBand)

输入Data为detrend后的time-series

	SamplePeriod 为采样周期即TR
    
	FreBand为滤波频段(0.01- 0.1 Hz)
Note：该方法虽然较为常用及快速，但其使用fft分解重建，滤波效果并非最优，可尝试使用Butterworth filter


3.	Regressout nuisance signals

Nuisance signals一般包含 headmotion (6个参数)，fMRI在WM里的均值，fMRI在CSF里的均值，fMRI全脑信息均值（可选），及以上8/9个值的diff），共16或18列数据，也可加入他们的平方*.^2, 构成32/36列数据

通过regression方法来regress out 16/18 /32/36列nusiance信号;

regress可参考[b,r,SSE,SSR, T, TF_ForContrast] = y_regress_ss(y,X,Contrast,TF_Flag)函数编写，该方法速度较快

5_swraFunImg:

配准到MNI空间后resample 并高斯平滑(FWHM=6mm)

T1Img: 结构像及分割结果

Rp_aFunImg_RI352_006.txt: realign过程产生的6个头动参数
