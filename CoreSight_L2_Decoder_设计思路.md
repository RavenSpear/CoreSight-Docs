# CoreSight_L2_Decoder设计
注：所有输入以最右为低位，根据L1的实现，数据可以有四个周期的时钟进行处理
## 组件1：PreProcessor

1. 输入

| 端口名 | 宽度 | 描述 |
| :----: | :---: | :-----------------: |
| trace_clk | 1 | 全局时钟 |
| in_data | 240 | 输入的ID_Data，为15x(8+8)的形式低8位为ID，高8位为Data |
| in_data_valid | 1 |输入数据可用 |
| in_ID | 8 | 当前L2_Decoder所属的PE的ID |

2. 输出

| 端口名 | 宽度 | 描述 |
| :----: | :---: | :-----------------: |
| out_cnt | 4 | 输出的数据中可用的数据数量 |
| out_R | 120 | 输出的数据，为8x(0-15), 可用的数据量由out_cnt指定 |
| PP_out_valid | 1 |输出数据可用 |

3. 周期行为

| 周期 | 行为 |
| :-: | :-: |
| 1 | 组合逻辑out_cnt随in_data_valid一同上升 |
| 2 | PP_out_valid上升，out_R产生数据 |

4. 用途

将输入的15个ID_Data进行过滤，得到其中符合当前L2_Decoder ID的数据，输出若干（0-15）纯数据序列。
输入的ID_Data存在两种形式，其一为DATA:ID，其二为ID:FF，后者为废弃数据，会被直接抛弃。

### 子组件1-1：Index_Formatter

1. 输入

| 端口名 | 宽度 | 描述 |
| :--: | :--: | :--: |
| IF_i_data_valid | 1 | 第i个数据是否是合法数据 |
| count | 15 | 低i位表示前i数据是否合法，高位用0填充 |

2. 输出

| 端口名 | 宽度 | 描述 |
| :----: | :---: | :-----------------: |
| IF_o_data | 4 | 计算出的合法数据存储的位置序数 |

3. 用途

根据输入的数据合法情况计算出每个数据在最终输出的位置序号

## 组件2：Collection

1. 输入

| 端口名 | 宽度 | 描述 |
| :----: | :---: | :-----------------: |
| trace_clk | 1 | 全局时钟 |
| C_i_data | 120 | 从PreProcessor中输入的数据  |
| in_valid_cnt | 4 | 可用数据量  |
| valid  | 1 | 输入数据可用  |

2. 输出

| 端口名 | 宽度 | 描述 |
| :----: | :---: | :-----------------: |
| C_o_data | 128 | Collection输出的完整的128位可用数据 |
| o_fifo_write | 1 | C_o_data已满，可写出 |

3. 周期行为

| 周期 | 行为 |
| :-: | :-: |
| 1 | data_valid在检测到C_i_valid上升后上升 |
| 2 | 如输出out_tmp_data已满，o_fifo_write上升，C_o_data产生数据 |

4. 用途

收集PreProcessor生成的数据流，组合成16个8位宽的连续数据流，写入FIFO。本组件中包含一个32x8的队列缓冲区，120位输入缓冲，128位输出缓冲。输入缓冲的数据直接进入缓冲区队列排队。判断队列长度，如果超过16，则将队首的16个数据输出到输出缓冲，后半部分的数据推至队首。

## 组件3：FIFO

1. 输入

| 端口名 | 宽度 | 描述 |
| :----: | :---: | :-----------------: |
| clk | 1 | 全局时钟 |
| F_i_data | 128 | 从Collection中输入的数据  |
| write_enable | 1 | 来自Collection的使能信号  |
| read_enable  | 1 | 来自ControlCore的读取使能信号  |

2. 输出

| 端口名 | 宽度 | 描述 |
| :----: | :---: | :-----------------: |
| out_valid | 1 | 输出使能 |
| F_o_data | 128 | 待ControlCore读取的数据|
| empty | 1 | FIFO为空 |shu

3. 周期行为

| 周期 | 行为 |
| :-: | :-: |
| X | read_enable的上升不可预知，一般read_enable会在empty为0下一个周期下降，out_valid上升，tmp_data生成数据 |
| 1 | 随着write_enable上升，mem从group_index处写入F_i_data, group_index+=1 |


4. 用途

收集Collection推送的128位数据，并将之排队；数据由ControlCore消费，ControlCore会判断当前FIFO是否为空，并通过read_enable发起读取请求，如out_valid为1，则数据会被ControlCore读取。