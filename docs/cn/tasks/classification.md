# 图像分类

## 1. 简述

图像分类任务中广泛使用了人工设计的深度学习网络，随着AutoML的研究的深入，特别是各种智能终端的出现，给网络设计提出了更高的要求，使得在图像分类任务中越来越多的使用自动学习来构建网络。

一般分类任务针对使用数据集的差异，有如下场景：

1. 使用公共数据集搜索和训练模型，验证和比较算法性能。
2. 有充分的私有训练数据，搜索和训练模型，用于特定的分类场景。
3. 使用少量的私有训练数据集，使用model zoo中的图像分类预训练模型，期望达到一个较高的模型性能。

对于如上三类场景，Vega都已提供了相应的支持。特别要提醒的是，在Vega中使用私有数据需要根据Vega的接口要求适配，可参考[数据集参考](../developer/datasets.md)

对于分类模型部署有如下约束：

1. 模型部署在云上，对模型的准确率有极致要求。
2. 模型部署在特定设备上，如手机上、使用DaVinci 310芯片的终端设备上，要求模型足够小，推理时间足够短，并且要满足一定的准确率。

Vega当前支持这两类约束，通过网络架构搜索等手段，生成多个模型，自动挑选出合适的模型供选择，同时支持在特定的硬件设备上评估模型性能，满足要求。

另外，针对算法研究科研人员，不满足于Vega提供的网络结构、搜索空间和搜索算法，希望通过自定义网络结构、搜索空间和搜索算法，搜索和训练分类模型。针对这类场景，Vega也提供了相应的支持。

## 2. 算法选择

针对图像分类的使用场景，可分为如下几种种使用场景：

![](./images/classification.png)

以下分别介绍这几种场景下，如何使用Vega来搜索和训练模型：

### 2.1 场景A：使用充足训练数据来搜索和训练部署在本地或云端的模型

这种场景只需要用户提供足够的训练数据，按照如下步骤配置流水线，即可完成模型的搜索和训练：

1. 使用NAS搜索算法搜索网络架构，输出备选网络。搜索算法可尝试使用BackboneNAS、CARS、DartsCNN三种之一，优先选择BackboneNAS算法，该算法基于ResNet，设置简单，可用于较大数据集。
2. 使用HPO确定模型训练参数，网络可从第一步NAS输出的网络中选择一个。这个步骤会输出网络训练超参。HPO算法可采用BOSS、BOHB、ASHA等，优先选择BOSS算法。
3. 使用数据增广算法来选择transform，推荐使用PBA算法。
4. 最后一步是fully train，将NAS搜索的备选网络、HPO搜索的超参、数据增广搜索出来transforms作为fully train的输入，训练一组模型，挑选出最优的模型。

具体的配置信息可参考[配置指导](../user/config_reference.md)

### 2.2 场景B：使用充足训练数据搜索和训练在终端上部署的模型

这种场景的关键在于在一定的约束条件下搜索模型，以方便部署在特定的终端上运行，步骤如下：

1. 使用NAS搜索算法搜索网络架构，输出备选网络。搜索算法可尝试使用BackboneNAS、CARS、DartsCNN三种之一，一般这种场景下模型较小，可优先选择CARS算法，该算法执行效率高，设置简单，适合于小模型的搜索。
2. 在NAS搜索阶段要考虑搜索出的模型在特定硬件上的性能，可配置评估服务，在模型搜索中将模型转换到特定硬件上进行评估其时延。（待提供）
3. 使用HPO确定模型训练参数，网络可从第一步NAS输出的网络中选择一个。这个步骤会输出网络训练超参。HPO算法可采用BOSS、BOHB、ASHA等，优先选择BOSS算法。
4. 使用数据增广算法来选择transform，推荐使用PBA算法。
5. 最后一步是fully train，将NAS搜索的备选网络、HPO搜索的超参、数据增广搜索出来transforms作为fully train的输入，训练一组模型，挑选出最优的模型。

具体的配置信息可参考[配置指导](../user/config_reference.md)

### 2.3 场景C：自定义搜索空间和搜索算法

这种场景下的步骤建议如下：

1. 使用NAS搜索算法搜索网络架构，输出备选网络。搜索算法可尝试使用BackboneNAS、CARS、DartsCNN三种之一，一般这种场景下模型较小，可优先选择CARS算法，该算法执行效率高，设置简单，适合于小模型的搜索。
2. 在NAS搜索阶段要考虑搜索出的模型在特定硬件上的性能，可配置评估服务，在模型搜索中将模型转换到特定硬件上进行评估其时延。（待提供）
3. 使用HPO确定模型训练参数，网络可从第一步NAS输出的网络中选择一个。这个步骤会输出网络训练超参。HPO算法可采用BOSS、BOHB、ASHA等，优先选择BOSS算法。
4. 使用数据增广算法来选择transform，推荐使用PBA算法。
5. 最后一步是fully train，将NAS搜索的备选网络、HPO搜索的超参、数据增广搜索出来transforms作为fully train的输入，训练一组模型，挑选出最优的模型。

### 2.4 场景C：使用预训练模型和少量训练数据搜索和训练在本地或云端使用的模型

这种场景下可考虑Vega的[Model Zoo](../model_zoo/model_zoo.md)中提供的`EfficientNet B8`模型，使用少量的训练数据进行fine tune，这里不再详述。

### 2.5 场景C：使用预训练模型和少量训练数据搜索和训练在终端使用的模型

这种场景下可考虑Vega的[Model Zoo](../model_zoo/model_zoo.md)中提供的`EfficientNet B8`模型，使用少量的训练数据进行fine tune，然后再考虑使用剪枝（待提供）和量化方法（待提供）将模型小型化，并使用模型评估服务进行模型评估，这里不再详述。

## 3. pipeline

在根据场景选择了算法后，下一步需要构造pipeline，配置pipeline配置文件，vega依据配置文件运行。
请参考[配置参考]完成pipeline的配置，在此不再详述。
