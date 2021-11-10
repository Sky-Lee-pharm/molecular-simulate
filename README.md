进行分子动力模拟至少需要以下四种文件：
1. 蛋白质数据库（pdb）文件，用于存储系统的原子坐标和速度。pdb文件可以自己生成，也可以通过网络获取，网址为：http://www.pdb.org。
2. 蛋白质结构文件（psf），用于存储蛋白质的结构信息，例如各种类型的键合相互作用。
3. 力场参数文件。力场是系统中原子所经历的势能的数学表达式。CHARMM、 X-PLOR、AMBER和GROMACS是四种力场，NAMD可以使用它们。
4. 一个配置文件，用户在其中制定NAMD在运行模拟时应采用的所有选项。配置文件告诉NAMD如何运行模拟。
 
生成蛋白质结构文件（PSF）
首先打开含有pdb的文件夹
展示删除1UBQ.pdb 中的水分子，并单独创建蛋白质pdb文件的过程
在终端输入vmd打开VMD
点击VMD的主窗口中的File-New molecule 菜单项加载1UBQ.pdb。在文件浏览器中找到1UBQ.pdb，按加载按钮加载文件。
值得注意的是：来自蛋白质数据库的X-射线结构不包含泛素的氢离子。这是由于X射线晶体学通常无法解析氢原子。
在VMD的主窗口中选择Extension-Tk console菜单项。并输入以下命令：
set ubq [atomselect top protein]	 
$ubq writepdb ubqp.pdb
在1-1-build文件夹下创建了ubqp.pdb，其中包含泛素的坐标，没有氢离子。
选择File-Quit 关闭VMD。
之后创建泛素的psf文件。值得注意的是，VMD可以通过点击Extensions-modeling-Automatic PSF Builder来进行创建psf文件。VMD的psfgen包在这方面非常有用。为了创建psf文件，首先要创建一个PGN文件，该文件将成为目标psfgen。
在终端中输入nedit打开文本编辑器，输入以下内容。
package require psfgen	 
topology top_all27_prot_lipid.inp	 
pdbalias residue HIS HSE	 
pdbalias atom ILE CD1 CD	 
segment U {pdb ubqp.pdb}	 
coordpdb ubqp.pdb U	 
guesscoord	 
writepdb ubq.pdb	 
writepsf ubq.psf
保存为ubq.pgn
刚刚创建的文件包含必要的命令，用于创建带有氢原子且不带水的泛素 psf 文件。pgn 文件的每个命令都有说明：
第 1 行：您将在 VMD 中运行psfgen。此行要求psfgen包可供 VMD 调用。
第 2 行：加载拓扑文件top_all27_prot_lipid.inp
第 3 行：将组氨酸的残基名称更改为拓扑文件中的正确名称。HSE 是组氨酸的三个名称之一，基于其侧基的质子化状态。
第 4 行：$\delta$异亮氨酸残基中名为“CD1”（碳）的原子被重命名为“CD”，这是拓扑文件中的专有名称。由于异亮氨酸仅包含一个$\delta$碳原子，因此 psf 文件不使用“CD”后的数字标签。
第 5 行：创建了一个名为 U 的段，其中包含来自ubqp.pdb 的所有原子。该段命令还增加了氢原子。
第 6 行：从ubqp.pdb中读取坐标，并匹配残基和原子名称。旧的段标签将被新的段标签“U”覆盖。
第 7 行：根据拓扑文件中的残基定义猜测缺失原子（如氢）的坐标。
第 8 行：写入包含所有原子（包括氢）的完整坐标的新 pdb 文件。
第 9 行：写入一个包含蛋白质完整结构信息的 psf 文件。
在终端输入以下命令
>vmd -dispdev text -e ubq.pgn
这样psf就创建成功了。

蛋白质溶剂化

蛋白质需要被溶剂化，即放入水中，以便更接近细胞环境。
1. 将蛋白包裹在真空的水球中，为没有周期性边界条件的最小化和平衡做准备。
2. 将蛋白包裹在水箱中，为周期性边界条件和平衡做准备。
用准备好的tcl脚本来创建水球。它称为wat_sphere.tcl，位于1-1 build文件夹中。
在终端中输入
vmd -dispdev text -e wat_sphere.tcl
这将调用脚本将蛋白包裹在尽可能小的水球中。wat_sphere.tcl的输出将是水球的中心及半径。记录这些数值。该脚本还创建了水球中的蛋白的pdb和psf文件ubp_ws.pdb，ubp_ws.psf。
终端输入exit退出VMD的文本版本。并输入vmd打开图形界面。
加载ubp_ws.pdb及ubp_ws.psf。
在VMD的Tk控制台中输入输入
package require solvate
solvate ubq.psf ubp.pdb -t 5 -o ubq_wb
Tk控制台会自动生成ubq_wb.psf和ubq_wb.pdb。
该命令会把蛋白包裹在一箱水中。-t后的选项为水箱的尺寸5a在每个方向上从具有最大原子层在该方向的坐标。
用vmd打开查看结构。
在Tk控制台输入
set everyone [atomselect top all]
measure minmax $everyone
分析系统中的所有原子，并提供最大值和最小值x, y 和z整个蛋白质-水系统的坐标。
用measure enter $everyone命令可以输出中心的坐标。
