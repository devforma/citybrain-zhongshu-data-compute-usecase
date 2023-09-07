# 使用citybrain中枢和计算平台来加速科研

这篇文章通过一个示例来介绍如何使用cityrain平台的数据和算力来加速科研工作者的研究进程，数据和算力这两大能力分别由citybrain两个核心服务：中枢和算力平台来提供支撑。


## 让我们开始吧

我们的 [citybrain](https://www.citybrain.org) 平台上托管着一些科研领域的知名数据，称之为 `Head Data` 。我们将这些原始数据处理成标准的表格型的数据并将其存储到citybrain计算平台中，该平台对外提供SQL接口来进行数据的查询和计算。这项工作使得原本面向存储的数据格式变身成面向计算了，我们认为这是发掘数据价值的第一步，也是非常重要的一步。

这次我们使用到了Head Data中的一个: [CHELSA](https://www.citybrain.org/#/start/dataset-detail?id=40400083), CHELSA(Climatologies at high resolution for the earth’s land surface areas) 提供了几十个年份的气象数据，内容为经纬度按月的降雨量、温度等信息。

我想要查下太平洋某个 `小岛A` 在1998年8月份的降雨数据，用于进一步的研究。我有一份geojson的数据，里面包含了`A岛`所在国家的所有岛屿的polygon列表，这个例子中我只关心 `A岛` 的数据，但也许未来的某天我像查下 `B岛`的数据或者把文件分享给其他人用来查询 `C岛`数据。有这样属性的数据在科研场景下是常见的，出于存储和传播的考虑，我决定将文件上传到citybrain平台。

打开citybrain.org网站并注册账户，登录后点击左侧边栏 `Datasets` 菜单进入个人数据集管理页面，在这里可以上传数据创建dataset。我的geojson文件并非csv格式，于是选择第二个标签，上传普通文件，该动作会创建新的dataset，包含了我们的文件。

上传完成后该数据集会在dataset列表中，点击右侧箭头会打开详情页面。在该页面 `Data access` 区域能看到我们上传的文件的 `Data Address`。

`Data Address`是citybrain中枢系统为每一个连接到中枢的data分配的唯一地址，平台用户上传的数据都会注册到中枢并分配这样一个地址。后续通过该地址可以便捷高效的访问数据，你也可以将该地址分享给其他用户。

## 好戏开场

目前为止，我们的实验准备工作已经完成，所需要的 `Head Data` 和个人上传的数据 （称之为`Longtail Data`）均已就绪，接下来开始编码了。

citybrain平台发布了python语言的package，使得用户可以用简单的方法调用来使用citybrain平台的数据和算力，该package github主页见 **[Citybrain Platform Python Library](https://github.com/citybrain-platform/python-library)**。

首先本地已安装python环境，终端或命令行中输入一下命令安装该package
```sh
pip3 install --upgrade citybrain-platform
```

安装完成后即可在python代码中使用该package来调用citybrain平台能力，本实现依次执行以下几个步骤

1. 使用该package需要先注册平台账户，通过网站的个人设置页获取该账号的apikey，该apikey用于使用python package时校验账户
    ```python
    import citybrain_platform

    citybrain_platform.api_key = "..."
    ```
2. 前面通过网站上传的geojson数据已经得到了中枢的data address，输入以下代码来访问该数据
    ```python
    citybrain_platform.Data.download(data_address="...", save_file="islands.geojson")
    ```
    该方法通过citybrain中枢系统来解析data address定位数据并将数据内容返回，保存成文件

3. 解析文件内容，提取 `A岛` polygon数据
    ```python
    # code to read islands.geojson and extract polygon data of island A
    ```
4. 构造SQL语句创建计算任务，该语句查询 `CHELSA` 这个 `Head Data` 下的chelsa_monthly表，查询条件为经纬度在`A岛` polygon范围内且日期为1988年8月的记录数
    ```python
    # the following code is just an example, it can't run
    sql = "SELECT avg(pr) FROM chelsa_monthly WHERE some_func(lat, lon) IN " + A岛polygon
    job_id = citybrain_platform.Computing.create_job(sql=sql)
    print(job_id)
    ```
5. 查询任务执行状态，任务完成后下载执行结果
    ```python
    from citybrain_platform import JobStatus

    job_status = citybrain_platform.Computing.get_job_status(job_id="...")
    
    if status.status == JobStatus.TERMINATED:
        citybrain_platform.Computing.get_job_results(job_id=job_id, filepath="result.csv")
    ```

该结果也可以通过网站上传，由中枢系统分配唯一data address，后续继续做科研或者分享给其他用户。
至此该实验已完成，其中涉及到数据和计算的部分分别体现了citybrain中枢系统和算力平台的使用，新的科研之路由此启程！
