# Prediction_Backend

 主要包括结构预测和活性预测模块的算法和后端。 

## Usage

### 1. 验证连接

#### 方法

Get

#### url

 http://20.106.156.143:4400/hello 

#### 请求参数

无

#### 返回

字符串hello

#### postman示例

![1633407840554](Prediction_Backend.assets/1633407840554.png)

### 2. pdb或fa/fasta文件转为蛋白质序列

#### 方法

Post

#### url

 http://20.106.156.143:4400/sequence

#### 请求参数

| 参数名 | 是否必须 | 类型                        | 说明                                                         |
| ------ | -------- | --------------------------- | ------------------------------------------------------------ |
| file   | 是       | 以.pdb/.fa/.fasta结尾的File | pdb或者fasta文件                                             |
| save   | 否       | string                      | 当save的内容为'target'时，会把文件存在target文件夹，返回一个result.tarPath，前端可以据此获取上传的file的url |

#### 返回

##### 正常返回

| 返回字段        | 字段类型 | 说明                                                         |
| --------------- | -------- | ------------------------------------------------------------ |
| result          | object   | 序列结果                                                     |
| result.id       | string   | 结果id，文件名                                               |
| result.seq      | string   | 结果序列，如果PDB文件中的序号不连续，中间插入X               |
| result.startPos | number   | 如果是pdb文件，则返回第0个残基(实际位置为0的残基)在PDB文件中的序号；如果是fasta文件，则返回1 |
| result.tarPath  | string   | 前端可以据此获取上传的file的url                              |

##### 错误返回

| 返回字段 | 字段类型 | 说明     |
| -------- | -------- | -------- |
| error    | string   | 错误信息 |

#### postman示例

##### 示例1

此示例中，pdb文件的残基从-3开始，所以结果的startPos返回-3。则如果有残基及其序号为SER433，则在链中的实际位置是433 - (-3) = 436。

![1633853356678](README.assets/1633853356678.png)

![1633853304966](README.assets/1633853304966.png)

##### 示例2

此示例中，pdb文件的残基从1开始，所以结果的startPos返回1。则如果有残基及其序号为GLU1，则在链中的实际位置是1 - (1) = 0。也即GLU1代表了返回的result.seq中的处于位置0的字母E。

![1633853702365](README.assets/1633853702365.png)

![1633853656223](README.assets/1633853656223.png)

##### 示例3

![1633966011019](README.assets/1633966011019.png)

获取文件的url:  http://20.106.156.143:4400/target/<tarPath>

![1633966159637](README.assets/1633966159637.png)

回车后弹出下载弹窗

![1633966119554](README.assets/1633966119554.png)

### 3. 结构预测

#### 方法

Post

#### url

 http://20.106.156.143:4400/structure

#### 请求参数

| 参数名   | 是否必须 | 类型   | 说明                                                         |
| -------- | -------- | ------ | ------------------------------------------------------------ |
| lightSeq | 是       | string | 光控蛋白序列                                                 |
| linker   | 是       | string | linker序列                                                   |
| tarSeq   | 是       | string | 目标蛋白序列(由用户提供)                                     |
| id       | 是       | string | 需唯一标识用户提交的结构预测任务，可以和用户id和提交时间等相结合 |

#### 返回

##### 正常返回

| 返回字段 | 字段类型 | 说明                                                     |
| -------- | -------- | -------------------------------------------------------- |
| task_id  | string   | 结构预测任务的id，可通过后面的任务查询方法查询任务的状态 |

##### 错误返回

| 返回字段 | 字段类型 | 说明     |
| -------- | -------- | -------- |
| error    | string   | 错误信息 |

#### postman示例

![1633852682712](README.assets/1633852682712.png)

### 4. 结构预测任务查询

#### 方法

Get

#### url

 http://20.106.156.143:4400/get/<task_id>

#### 请求参数

| 参数名  | 是否必须 | 类型   | 说明                            |
| ------- | -------- | ------ | ------------------------------- |
| task_id | 是       | string | 调用结构预测方法时返回的task_id |

#### 返回

| 返回字段                              | 字段类型                          | 说明                                                       |
| ------------------------------------- | --------------------------------- | ---------------------------------------------------------- |
| info                                  | object                            | 任务未完成时为null，完成时为一个包含获取结果的方法的object |
| status                                | string (PENDING，FAILED，SUCCESS) | literally                                                  |
| info.TM_score                         | object                            | 包含五个结果模型的TM score信息                             |
| info.TM_score.model\_\<num>           | string                            | 存放结果模型对应的分数                                     |
| info.pdbId                            | string                            | 存放结果的文件夹                                           |
| info.result_paths                     | string                            | 包含结果文件名信息的对象                                   |
| info.result_paths.TM_score_file       | string                            | 存放五个结果模型的TM score的文件                           |
| info.result_paths.model\_\<num>\_file | string                            | 存放结果模型的pdb文件                                      |
| info.error                            | string                            | 长任务运行过程中如果出现error，会由此字段显示错误信息。    |

#### postman示例

##### 任务未完成

![1633427567759](Prediction_Backend.assets/1633427567759.png)

##### 任务已完成

![1633853006747](README.assets/1633853006747.png)

![1633853160163](README.assets/1633853160163.png)

#### 浏览器示例

##### 任务已完成

![1633853109610](README.assets/1633853109610.png)

### 5. 获取结构预测结果文件

#### 方法

Get

#### url

 http://20.106.156.143:4400/results/<pdbId\>/\<name\>

#### 请求参数

| 参数名 | 是否必须 | 类型   | 说明                                                      |
| ------ | -------- | ------ | --------------------------------------------------------- |
| pdbId  | 是       | string | 调用结构预测任务查询时返回的info.pdbId                    |
| name   | 是       | string | 调用结构预测任务查询时返回的info.result_paths.\<filename> |

#### 返回

给定url下的文件。

#### postman示例

![1633429012146](Prediction_Backend.assets/1633429012146.png)

#### 浏览器示例

直接弹出下载弹窗

![1633429070231](Prediction_Backend.assets/1633429070231.png)

### 6. 可能活性位点的获取

#### 方法

Post

#### url

 http://20.106.156.143:4400/active/residual

#### 请求参数

| 参数名 | 是否必须 | 类型             | 说明                                                         |
| ------ | -------- | ---------------- | ------------------------------------------------------------ |
| file   | 是       | 以.pdb结尾的File | pdb文件，正常来说，应该是结构预测步骤中输入的光控蛋白(lightSeq)或者目标蛋白(tarSeq)的pdb，目标蛋白是用户输入的蛋白，而光控蛋白用户可以在前面的步骤(4.1.2 choose opto-switches)中获取。 |

#### 返回

##### 正常返回

| 返回字段           | 字段类型 | 说明                                                         |
| ------------------ | -------- | ------------------------------------------------------------ |
| result             | object   | 活性位点结果                                                 |
| result.active_list | list     | 其元素为表示活性位点的字符串，氨基酸三字符缩写+在蛋白质中的位置。用户可以在前端选择其感兴趣的位点，作为下面活性预测方法的参数，获取相应活性位点的活性预测分数。 |
| result.srcPath     | string   | 用户上传的PDB文件的路径，用于7. 活性预测的输入               |
| result.seq         | string   | 和2. pdb或fa/fasta文件转为蛋白质序列中的result.seq相同       |
| result.startPos    | number   | 和2. pdb或fa/fasta文件转为蛋白质序列中的result.startPos相同  |

##### 错误返回

| 返回字段 | 字段类型 | 说明     |
| -------- | -------- | -------- |
| error    | string   | 错误信息 |

#### postman示例

![1633960655807](README.assets/1633960655807.png)

### 7. 活性预测

#### 方法

Post

#### url

 http://20.106.156.143:4400/active/cad

#### 请求参数

| 参数名      | 是否必须 | 类型             | 说明                                                         |
| ----------- | -------- | ---------------- | ------------------------------------------------------------ |
| srcPath     | 是       | string           | 6. 可能活性位点的获取中返回的result.srcPath，正常来说，应该是结构预测步骤中输入的光控蛋白(lightSeq)或者目标蛋白(tarSeq)的pdb在第6步中上传后的存放路径。 |
| tarPath     | 否       | string           | 4. 结构预测任务查询中可以获取的文件url，正常来说，应该是存放结构预测返回的融合蛋白(光控蛋白+linker+目标蛋白)pdb的路径(info.pdbId + '/' + info.result_paths.model\_\<num>\_file) |
| tarPDB      | 否       | 以.pdb结尾的File | 若没有tarPath参数，则取此处上传的pdb作为融合蛋白计算cad。    |
| active_list | 是       | list             | 用户从6. 可能活性位点的获取中得到的result中挑选出感兴趣的位点的列表。 |
| thresh      | 否       | number           | 计算活性位点cad选取的半径，默认为4                           |

#### 返回

##### 正常返回

| 返回字段                    | 字段类型 | 说明                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| result                      | object   | 包含不同活性位点的cad结果。如果用户选取的半径过小，可能不会包含输入的active_list中的某些活性位点。 |
| result.<active_residual>    | object   | 包含活性位点(也即活性残基)的cad分数。                        |
| result.<active_residual>.AA | float    | 包含活性位点(也即活性残基)的AA类型的cad分数。                |
| result.<active_residual>.AS | float    | 包含活性位点(也即活性残基)的AS类型的cad分数                  |
| result.tarPath              | string   | 若没有tarPath参数，则此处返回请求中上传的tarPDB的存储路径，可用于8. 获取活性预测上传的文件中获取tarPDB的url。 |

##### 错误返回

| 返回字段 | 字段类型 | 说明     |
| -------- | -------- | -------- |
| error    | string   | 错误信息 |

#### postman示例

##### 示例1：用户选择模型

![1633866408433](README.assets/1633866408433.png)

![1633866762575](README.assets/1633866762575.png)

##### 示例2：用户上传模型

![1634643663538](README.assets/1634643663538.png)

![1634643869826](README.assets/1634643869826.png)

### 8. 获取活性预测上传的文件

#### 方法

Get

#### url

 http://20.106.156.143:4400/active/<path>/<name>

#### 请求参数

| 参数名 | 是否必须 | 类型   | 说明                                                         |
| ------ | -------- | ------ | ------------------------------------------------------------ |
| path   | 是       | string | 调用6. 可能活性位点的获取时返回的result.srcPath中/之前的部分，或者7. 活性预测返回的result.tarPath中/之前的部分。 |
| name   | 是       | string | 调用6. 可能活性位点的获取时返回的result.srcPath中/之后的部分，或者7. 活性预测返回的result.tarPath中/之后的部分。 |

#### 返回

给定url下的文件。

#### 浏览器示例

![1633961125189](README.assets/1633961125189.png)

回车后直接弹出下载弹窗。

![1633961096300](README.assets/1633961096300.png)



### 9. 活性预测退出

#### 方法

Post

#### url

 http://20.106.156.143:4400/active/quit

#### 请求参数

| 参数名  | 是否必须 | 类型   | 说明                                                         |
| ------- | -------- | ------ | ------------------------------------------------------------ |
| srcPath | 是       | string | 6. 可能活性位点的获取中返回的result.srcPath，正常来说，应该是结构预测步骤中输入的光控蛋白(lightSeq)或者目标蛋白(tarSeq)的pdb在第6步中上传后的存放路径。 |

#### 返回

字符串success

#### postman示例

![1633867213502](README.assets/1633867213502.png)

### 10. 序列优化

#### 方法

Post

#### url

 http://20.106.156.143:4400/optimize 

#### 请求参数

| 参数名       | 是否必须 | 类型   | 说明                                       |
| ------------ | -------- | ------ | ------------------------------------------ |
| seq          | 是       | 字符串 | 需要优化的蛋白质序列                       |
| species_list | 是       | list   | 物种列表                                   |
| weights_list | 是       | list   | 每个物种对应的权重列表，必须和物种一一对应 |

#### 返回

##### 正常返回

| 返回字段             | 字段类型 | 说明                                               |
| -------------------- | -------- | -------------------------------------------------- |
| result               | object   | 优化序列结果                                       |
| result.optimmized_ad | string   | Optimized Sequence: Sequence (Squared Difference)  |
| result.optimmized_sd | string   | Optimized Sequence: Sequence (Absolute Difference) |
| result.peptide_seq   | string   | Translated Sequence: Peptide Sequence              |

##### 错误返回

| 返回字段 | 字段类型 | 说明     |
| -------- | -------- | -------- |
| error    | string   | 错误信息 |

#### postman示例

![1633961225762](README.assets/1633961225762.png)

![1633961240890](README.assets/1633961240890.png)