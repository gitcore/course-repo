# Lin's First Game — 六年级原创编程启蒙读本

> 原创虚构故事，不是教材课文，也不是某一款软件的操作说明。孩子可以先用纸笔完成页末挑战，再在熟悉的图形编程工具中尝试。插画中的积木是教学示意，不代表软件界面截图。

## 阅读目标与边界

- **阅读目标：** 读懂 Lin 怎样把想法变成小游戏；用 `first`、`then` 说出步骤，用 `repeat` 解释重复动作，用 `test` 和 `fix` 讲清尝试与修正。
- **编程目标：** 从一个很小的目标开始：按下方向键，让小猫沿路碰到星星。事件积木在按键时启动动作，移动积木放在重复积木内执行多次；根据运行结果调整次数。
- **教学边界：** 正文是原创情境，借用图形编程常见概念，不要求六年级学生会写文本代码，也不声称机器会自己思考。

编排参考：[Code.org Express Course 的年级范围](https://code.org/en-US/curriculum)、[Express Course 的顺序、调试、循环与项目单元](https://studio.code.org/courses/express-2025/units/1)、[Scratch Foundation 对调试的说明](https://www.scratchfoundation.org/learn/learning-library/debugging)、[Scratch Foundation 的创造性学习方法](https://www.scratchfoundation.org/learn/learning-library/scratch-creative-learning-philosophy)。这里的故事与活动是依据这些入门概念作出的原创教学设计，不是上述机构指定的六年级课程。

## 四页英文正文与中文辅助

### 01 · A small idea

1. Last week, Lin wanted to make a small game.（上周，Lin 想做一个小游戏。）
2. “Can I make a cat catch a star?” she asked.（“我能让小猫去抓星星吗？”她问。）
3. Her teacher showed her colourful coding blocks on a screen.（老师在屏幕上给她看了彩色的编程积木。）
4. Lin drew the cat, the star and a path on paper.（Lin 在纸上画出了小猫、星星和一条路线。）

### 02 · First, then

5. First, Lin chose a “when key pressed” block.（首先，Lin 选了一个“按键时启动”的积木。）
6. Then she joined a “move” block under it.（然后，她在下面接上一个“移动”积木。）
7. She pressed the key, and the cat moved a little.（她按下按键，小猫向前移动了一点。）
8. Before each new test, she put the cat back at the start.（每次重新测试之前，她都把小猫放回起点。）

### 03 · Try and fix

9. But the cat stopped before it reached the star.（但是小猫在碰到星星之前就停下了。）
10. Lin put a “repeat” block around the “move” block.（Lin 把“移动”积木放进了“重复”积木里。）
11. She set it to five and pressed the key again.（她把次数设为五，再次按下按键。）
12. The cat went too far, so Lin changed five to three.（小猫走得太远了，于是 Lin 把五改成了三。）

### 04 · A new level

13. She tested it again, and the cat reached the star.（她又试了一次，这回小猫碰到了星星。）
14. Lin added a happy sound and showed the game to a friend.（Lin 加入一段欢快的声音，并把游戏给朋友看。）
15. “I found a problem and fixed it,” Lin said.（“我发现了问题，还把它改好了，”Lin 说。）
16. “Next week, I’ll make a new level!”（“下周，我要做一个新关卡！”）

## 六年级英语知识点连接

- **六上 Unit 1 · Try your best：一般过去时与继续尝试。** `Last week` 引出已发生的制作过程；`wanted`、`showed`、`pressed`、`changed`、`tested` 是规则动词过去式，`drew`、`chose`、`put`、`went` 是不规则形式。故事中的先尝试、再调整与 `Have a go!`、`Keep trying!` 的主题相呼应，但不是教材故事。
- **六上 Unit 5 · Keeping our city clean：will 表达计划。** 最后用 `Next week, I’ll make a new level` 说将来的计划；这里迁移的是语法，不是该单元的环保内容。
- **阅读顺序词：** `First`、`Then`、`again`、`before` 帮助孩子复述步骤与结果。这里把编程中的“按顺序执行”变成可看见的故事动作。

## 阅读支架

| 词语 | 提示 |
|---|---|
| coding blocks | 图形编程积木；把命令拼接起来 |
| path | 路线 |
| when key pressed | 按下指定按键时启动 |
| move | 移动 |
| repeat | 重复执行里面的动作 |
| test | 运行看看是否达到目标 |
| fix | 找到原因后修改 |
| level | 游戏关卡 |

## 读后动手试一试

1. **纸上排步骤。** 画出小猫和三格外的星星，先写“按键启动”，再写“向前走一格”。如果只执行一次，小猫会在哪里？
2. **预测与重复。** 每次测试前，都把小猫放回同一个起点。把“向前走一格”放进“重复 3 次”里。按键一次后，小猫会在哪里？回到起点，把次数改成 5，又会怎样？
3. **发现与修正。** 如果目标在第三格，却设置了“重复 5 次”，把哪一个数字改掉？请用“我先预测……，运行后发现……，所以修改……”说出理由。
4. **做自己的版本。** 换一个角色或目标，并请同伴先预测运行结果，再试玩。想增加新关卡时，一次只改一个条件，再测试是否符合预期。

读后核对：① Lin 先在纸上画了什么？小猫、星星和路线。② 哪块积木负责按键时启动？`when key pressed`。③ 为什么要把五改成三？小猫走过了星星，改为三次移动才到达目标。④ Lin 下周计划做什么？一个新关卡。
