# Gromacs测试算例  

包含一些Gromacs性能分析测试算例，具体内容如下：  
  
## waterbox  
  
随机生成的水分子盒子，不包含蛋白质分子。可以通过调节模拟盒子大小，生成不同大小的测试算例。  
  
#### 算例构建步骤（采用Gromacs2018.4构建）  
  
本节记录了waterbox算例构建方法，供后续生成新算例参考。算例只需构建一次，后续性能评估测试使用生成的.tpr文件即可。  
构建步骤：  
1）拷贝一个空的结构文件conf.gro,文件内部包含模拟盒子大小，本体系为 12x12x12 nm  
2）拷贝一个合适的力场文件topol.top,删除[ molecules ]行后面的内容。  
3）在模拟盒子中随机生成水分子：  
gmx solvate -cp conf.gro -cs spc216.gro -o conf2.gro -p topol.top  
4）拷贝能力能量化配置文件em.mdp,并生成能里最小化输入：  
gmx grompp -f em.mdp -p topol.top -c conf2.gro -o em.tpr  
5）进行能量最小化弛豫  
gmx mdrun -v -ntmpi 1 -ntomp 4 -gpu_id 0 -deffnm em  
6）拷贝nvt弛豫文件，并生成算例输入：  
gmx grompp -f nvt.mdp -p topol.top -c em.gro -o nvt.tpr  
7）进行nvt弛豫：  
gmx mdrun -v -ntmpi 1 -ntomp 4 -gpu_id 0 -deffnm nvt  
  
经过以上步骤之后，即可构建一个合理的算例。算例包括一下文件：  
nvt.gro  分子位置坐标  
topol.top  拓扑文件  
后续可以选取合适的分子动力学配置文件，进行实际的算例测试。  
  
Gromacs中配置文件为 .mdp文件，所有配置选项都在此文件中。  
  
#### 构建waterbox PME算例  
1）拷贝配置文件md_pme.mdp  
2）生成算例输入：  
gmx grompp -f md_pme.mdp -p topol.top -c nvt.gro -o md_pme.tpr  
3）算例测试：  
gmx mdrun -v -ntmpi 1 -ntomp 4 -gpu_id 0 -deffnm md_pme  
  
#### 构建waterbox cutoff算例(关闭PME)  
1）拷贝配置文件md_cutoff.mdp  
2）生成算例输入  
gmx grompp -f md_cutoff.mdp -p topol.top -c nvt.gro -o md_cutoff.tpr  
3）算例测试：  
gmx mdrun -v -ntmpi 1 -ntomp 4 -gpu_id 0 -deffnm md_cutoff  
  
#### 性能测试说明  
关闭PME的测试使用md_cutoff.tpr输入文件即可  
  
分子动力模拟是显示积分求解过程，每步积分的时间都是固定的。积分步长在 .mdp配置文件中。  
如果想延长或缩短算例测试时间，可以修改相应的积分步长，方法如下：  
  
修改积分步长方法：  
1）编辑相应的 .mdp文件，nsteps参数为积分步，修改此参数  
nsteps                  = 50000  
2）注意！！！，修改 .mdp文件后一定要重新生成算例输入 .tpr文件  
gmx grompp -f md_cutoff.mdp -p topol.top -c nvt.gro -o md_cutoff.tpr  
3）用新生成的 .tpr文件进行模拟测试：  
gmx mdrun -v -ntmpi 1 -ntomp 4 -gpu_id 0 -deffnm md_cutoff  







