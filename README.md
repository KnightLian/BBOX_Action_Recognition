###### Read in Typora

## 行为识别模型 Behavior Recognition Network 

### Data

- 数据获取于 yolo v5 对 小学生化学实验时 所做动作的视频执行目标检测的结果
- 数据分为正负样本，行为分为上试管夹、固体缓慢竖起、搅拌
- 每段视频数据包含以上的一个单一行为，期望的测试样本为47个以上行为的组合动作
- 样本提供拍摄时间和摄像头位置，数据是每一帧的物体被检测到的 yolo 框信息
- yolo 框信息包括：被检测到的物体的标签、信心得分、中心坐标在x轴、中心坐标在y轴、以及物体的宽和高
- 被检测到的物体包括：手、试管、试管夹子、玻璃棒、烧杯

### 数据处理 create dataset

- shuffle 单一行为的视频数据，将每一帧里的信心得分低于0.7的所有数据删掉，处理重影、标错等
- 设计模板：手、手、试管、试管夹子、玻璃棒、烧杯。将每一帧数据按对应位置放入，若没有就填-1，数据清洗使其统一化
- 对被检测到的物体进行独热编码，再 drop 掉信心得分栏减少数据量，再 flatten 每帧数据、将以上6个模板物体信息压成一行
- 单一行为的视频数据在被处理后变为 tensor，其大小为 Fx54，F 为视频长度，54代表6个物体信息
- 按8：2将单一行为的视频数据分开，来创建训练集和验证集，将给定的47个组合行为的视频数据定义为测试集
- 分别对分开后的单一行为的视频数据进行抽样，抽取2个到4个再按 F 维度拼接，增广数据并合成4000个训练数据和400个验证数据
- 每段单一行为的视频数据的标签也随之合并，组合行为创建的规则可能与测试集的不同，合成的行为没有2个动作之间的过渡状态

### Model

- 不定长 F 视频长度被放在第三个 height 维度上，不能放在第一和第二个 batch 或 channel 维度
- 争取不让网络太深，参照 yolo v5 的 CBL 模块，加入正则项和 dropout，调参其最优
- 模型构成: 8层 CNN + 1层双向 LSTM + 4层 FC
- 采用 CTC loss 处理不定长问题，以及 adam 来做优化
- 克服过拟合，提升学习能力，以及绕过算法无法并行而导致训练速度慢，最高验证集精度95%，测试集精度50%

