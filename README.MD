# 基于java的俄罗斯方块小游戏
# 打开方式及环境
使用Eclipse运行即可打开 
## 类之间的关系图
（注：折线箭头仅代表后者为前者的一个实例，弯箭头为继承，双向箭头为相关。）
![1.png](https://i.loli.net/2019/12/15/yP3j5ChQROurli2.png)
 

## 各个类说明
（注：红色的为类名，黄色的为该类中比较重要的实例对象。）

1．	程序入口：Main
main方法里弹出选择提示，选择单人模式则进入singleGame函数，创建GameFrame类进行游戏（详见第2条）；
选择双人模式则进入doubleGame函数，创建DoubleGameFrame类（继承自GameFrame类）进行游戏（详见第3条）。

2．	单人模式框架类：GameFrame
继承自JFrame类，包含两个主体部件GameView和WatchView（详见第4条和第5条）；
1.构造方法设置布局，为WatchView的对象watch的JButton添加鼠标监听器控制游戏开始和暂停，添加键盘监听器调用GameView的对象game的方法控制方块，以及计时器timer调用game中的方法控制方块下落和游戏的开始暂停；
2.方法updateData在方块落致底部时由timer调用，令game和watch更新游戏数据（如消去行，提取一个新的下落方块，增加得分，通过关卡，改变timer的延时加快方块下落速度等），返回值为消去的行数；
3.方法resetGame在每次开始新游戏之前对数据进行重置；
4.方法restartGame在暂停时恢复游戏，否则开始新游戏；
5.方法pauseGame暂停游戏；
6.方法stopGame在游戏结束时弹出提示信息。

3．	双人模式网络通信框架类：DoubleGameFrame
继承自GameFrame类，在此基础上又新增一个主部件CommuicateView（详见第6条）；
1.构造方法使用给定的ip和发送及接受端口建立网络通信，设置布局，建立四个线程sendThread、acceptThread、sendDataThread、acceptDataThread通过输出输入流及ImageIO分别发送、接受战况视图，通过输入输出流发送、接受战况数据（对方是否消去多重行数）；
2.方法creatView通过截屏创建己方战况视图并返回；
3.重写父类的方法updateData、resetGame、restartGame、pauseGame、stopGame，取消了开始暂停功能，新增winGame方法，赢得游戏则进入winGame弹出提示信息，否则进入stopGame弹出提示信息。

4．	游戏视图类： GameView
继承自JPanel类，通过此类操纵并显示游戏中的方块，出现在框架的最左边；
1.构造方法设置格式布局，以绿色背景的JLabel数组box填充，nextUnit列表存储后续的Unit方块（详见第7条），movingUnit是正在移动的Unit方块；
2.方法romoveRow消去可消去的行并返回消去数量、addRow增加指定行数；
3.方法creatNextUnit向nextUnit中添加后续的方块Unit；
4.方法getNextUnit从nextUnit中取出并试着绘制新方块movingUnit，若无法绘制，返回false，说明方块已落至顶部、游戏结束；
5.方法moveUnit根据给定的指令变换或移动正在活动的方块movingUnit，在方块落至底部的时候返回false；
6.方法paintUnit被getNextUnit和moveUnit所调用，通过改变JLabel的透明属性显现方块，试着在给定位置重新绘制正在移动的movingUnit，若超出边界或与已有方块重合则不绘制，返回false；
7.方法resetGame和endGame分别完成界面的重置、停止所有方块移动。

5．	数据视图类： WatchView
继承自JPanel类，此类中包含数据游戏得分score、游戏关卡level，可观察当前游戏得分、关卡、下一个方块，可通过开始和暂停按钮开始或暂停游戏，出现在框架的中间；
1.构造方法设置布局；
2.方法resetData在新一轮游戏开始时重置数据；
3.方法addStartListener和addPauseListener分别为开始和暂停按钮添加监听器；
4.方法updateData更新面板上的分数、关卡、下一个方块的信息；
5.方法getLevel为外部传递当前关卡（主要用于设置方块下落速度）。

6．	通信视图类： CommunicateView
继承自JPanel类，此类实时显示对方的游戏战况，只在双人模式中有，出现在框架最右边；
1.构造方法设置布局；
2.方法setImage更新数据域对方战况img，并调用提供的重绘方法repaint；
3.重写方法paint，设置绘图区域与放缩，被repaint方法调用。

7．	方块类： Unit
此类中包含一个点location描述方块的位置，以及一个int数组key来描述方块的种类及形状（详见第8条）；
1.两种构造方法，一种是在指定位置生成一个随机形状的Unit，另一种是用提供的key在指定位置生成指定形状的Unit；
2.方法getShape返回此Unit的形状绘制在坐标系中的坐标点数组(WatchView使用)；
3.方法getChangedUnit返回此Unit经过变形或位移产生的新Unit；
4.方法getPaintLocation返回此Unit的绘制在坐标系中的坐标点数组（GameView使用）。

8．	方块键值类： UnitKey
此类中list包含了所有方块形状绘制在坐标系中的点的数组；
1.方法getRandomKey产生一个随机的key值返回，key值对应着某个方块的形状；
2.方法getNextKey返回给定key值的方块经变形后对应的key值；
3.方法getShapeByKey解读给定key值对应的方块的形状，以Point数组的形式返回形状。

## 	程序使用说明（文字说明，并附界面抓图）
1.	若要进行双人模式,可改变Main类中doubleGame的IP地址和端口号（设定的两端口号之差需大于1）。按（对方IP，对方接受端口，己方接受端口）的顺序设置。
示例：A的设置（B的IP，B接受端口,A接受端口）
B的设置（A的IP，A接受端口，B接受端口）
2.	直接运行程序，选择游戏模式。
	
![2.png](https://i.loli.net/2019/12/15/fOciypS3CEM81RZ.png)
  
3.	单人模式下可选暂停、开始，在非暂停时候点开始则直接开始新游戏。点击开始后通过方向键左右上下控制方块移动、变形、加速。右侧可看到下一个方块以及当前分数、关卡，每过1000分升一级，下落速度加快三分之一。游戏结束时提示最终得分，点开始开启新游戏。

![3.png](https://i.loli.net/2019/12/15/GLDKsPWb9u4Qk62.png)

![4.png](https://i.loli.net/2019/12/15/K4jzMWIb6YVrpgq.png)
         
4.	双人模式下Main中提前设置好后就可以直接开始游戏，此时开始暂停功能已禁用，在单人模式基础上增加了观察对方战况的区域（有可能因为对方屏幕视图读写异常而花屏或卡着，暂不知道怎么处理这种情况）。若一次消去多行，对方便会涨一行。直至一方输掉游戏，游戏结束，分别提示双方是输是赢。
![5.png](https://i.loli.net/2019/12/15/kqHVJia47rmne6O.png)

![6.png](https://i.loli.net/2019/12/15/GgpafM5PxZLkhdo.png)

![7.png](https://i.loli.net/2019/12/15/16TnUGpkbYhSZzg.png)
    
