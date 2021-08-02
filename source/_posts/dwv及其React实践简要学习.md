---
title: DWV及其React实践简要学习
tags:
  - React
  - DWV
categories:
  - - Front-end
date: 2021-07-23 16:53:35
---
# DWV简介
>DWV (DICOM Web Viewer) is an open source zero footprint medical image viewer.

DWV是一款开源的Web端DICOM医学影像查看器，这是其 [官方文档](https://ivmartel.github.io/dwv/doc/stable/index.html)。

从其架构上来看：
![DWV项目架构](architecture-overview.png)

拆分为两个过程：一是**load**，对DICOM数据的加载；二是**view**，对视图的渲染，包括图层控制（视图层-对图片渲染展示、绘制层-利用Konva进行标注）和工具控制（控制工具的加载和派发交互事件到已选工具）两部分。

# React实践
本次主要学习如何对项目进行组织并应用到React框架上，因此重点关注其在[React上的实践](https://github.com/ivmartel/dwv-react)

可以很方便地下载源码并运行起来这个小的Demo，具体效果是将DICOM文件拖入区域，读取数据后渲染出一幅图像，并利用工具进行操作。

![运行结果](2021-07-23-17-09-37.png)

![运行结果](2021-07-23-17-12-32.png)

在控制台看DOM元素的结构，标注信息层和图像层是分开到2个canvas中的：

![DOM元素结构](2021-07-23-17-41-04.png)

接下来看源码，结构如下：
![源码结构](2021-07-23-17-13-55.png)

标准的React项目，UI界面框架用的是`Material-UI`，现在主要看`DwvComponent`组件中的标尺如何进行实现。

### 构造函数
```JavaScript
constructor(props) {
    super(props);
    this.state = {
      // 版本控制
      versions: {
        dwv: dwv.getVersion(),
        react: React.version
      },

      // 工具设置
      tools: {
        Scroll: {},
        ZoomAndPan: {},
        WindowLevel: {},
        Draw: {
          options: ['Ruler'], // 声明绘制工具的“选项”为“标尺”
          type: 'factory',
          events: ['drawcreate', 'drawchange', 'drawmove', 'drawdelete'] // 包括4个响应事件
        }
      },
      toolNames: [],
      selectedTool: 'Select Tool',
      
      // 数据加载相关
      loadProgress: 0,
      dataLoaded: false,
      dwvApp: null,
      metaData: [],
      showDicomTags: false,
      
      // 工具在菜单上挂载元素
      toolMenuAnchorEl: null,

      dropboxDivId: 'dropBox',
      dropboxClassName: 'dropBox',
      borderClassName: 'dropBoxBorder',
      hoverClassName: 'hover'
    };
  }
```
在构造函数里直接用state来控制图片数据加载和工具设置

### 渲染函数
```JavaScript
render() {
    const { classes } = this.props;
    const { versions, tools, toolNames, loadProgress, dataLoaded, metaData, toolMenuAnchorEl } = this.state;

    // 将工具名称通过一个工具控制函数绑定，并返回包括工具的子菜单，toolNames从state中获取
    const toolsMenuItems = toolNames.map( (tool) =>
      <MenuItem onClick={this.handleMenuItemClick.bind(this, tool)} key={tool} value={tool}>{tool}</MenuItem>
    );

    return (
      <div id="dwv">
        <LinearProgress variant="determinate" value={loadProgress} />
        <div className="button-row">
          <Button variant="contained" color="primary"
            aria-owns={toolMenuAnchorEl ? 'simple-menu' : null}
            aria-haspopup="true"
            onClick={this.handleMenuButtonClick}
            disabled={!dataLoaded}
            className={classes.button}
            size="medium"
          >{ this.state.selectedTool }
          <ArrowDropDownIcon className={classes.iconSmall}/></Button>
          <Menu
            id="simple-menu"
            anchorEl={toolMenuAnchorEl}
            open={Boolean(toolMenuAnchorEl)}
            onClose={this.handleMenuClose}
          >
            {toolsMenuItems}
          </Menu>

          <Button variant="contained" color="primary"
            disabled={!dataLoaded}
            onClick={this.onReset}
          >Reset</Button>

        <!-- 省略了tags对话框的DOM -->

        <div id="dropBox"></div>

        <!-- 这里存放两个图层，一个为图像层，一个为标注层 -->
        <div className="layerContainer"></div>

        <!-- 省略了页脚的DOM -->

      </div>
    );
  }
```

### 生命周期函数
`componentDidMount`函数会在`render`渲染完之后执行，此时组件已经初始化完毕。
```js
componentDidMount() {
    // create app
    var app = new dwv.App();
    // initialise app
    app.init({
      "containerDivId": "dwv",
      "tools": this.state.tools
    });

    // load events
    let nLoadItem = null;
    let nReceivedError = null;
    let nReceivedAbort = null;

    // 通过addEventListener函数向类dwv.App中添加事件监听
    app.addEventListener('loadstart', (/*event*/) => {
      ...
    });
    // 在图片加载后，将工具名称添加到state中
    app.addEventListener("load", (/*event*/) => {
      // set dicom tags
      this.setState({metaData: dwv.utils.objectToArray(app.getMetaData())});
      // available tools
      let names = [];
      for (const key in this.state.tools) {
        if ((key === 'Scroll' && app.canScroll()) ||
          (key === 'WindowLevel' && app.canWindowLevel()) ||
          (key !== 'Scroll' && key !== 'WindowLevel')) {
          names.push(key);
        }
      }
      this.setState({toolNames: names});
      this.onChangeTool(names[0]);
      // set the selected tool
      let selectedTool = 'Scroll'
      if (app.isMonoSliceData() && app.getImage().getNumberOfFrames() === 1) {
        selectedTool = 'ZoomAndPan';
      }
      this.onChangeTool(selectedTool);
      // set data loaded flag
      this.setState({dataLoaded: true});
    });
    app.addEventListener('loadend', (/*event*/) => {
      ...
    });
    
    ...
    
    // handle window resize
    window.addEventListener('resize', app.onResize);

    // store
    this.setState({dwvApp: app});

    // setup drop box
    this.setupDropbox(app);

    // possible load from location
    dwv.utils.loadFromUri(window.location.href, app);
  }
```

### 控制函数
此步骤将一些dom操作和事件操作封装为函数放到整个组件里。

### 标尺工具
在dwv中留有class类的接口进行实现，内部封装的是Konva，找源码看：

![标尺实现](2021-07-23-18-21-37.png)

应该是将多个Konva.Line放到一起组成标尺的基本形状，至于长度的计算，思路应该是根据两个端点的坐标求出像素长度，再根据图片的比例换算为实际长度。

# 总结

- DWV的结构的重点在于，把工具的添加实现了自动化，只需要在state中声明需要的工具，通过渲染函数和dwv.app类的接口调用即可实现工具的添加
- 对于图像的查看功能实现，应按照逻辑进行分层来处理：图形层（获取数据并显示）和绘制层（形状标注），对于事件的监听放到最外层，以避免层覆盖使得事件监听失效。同时可以仿照DWV，对事件监听的控制统一进行添加管理
- DWV这个Demo较小，对于数据流没有明显展现，如果使用Mobx数据流的话，可以将**数据获取**和**界面状态**分别封装为两个Store

目前只是对dwv简单进行了解，其接口使用、具体实践还有待进一步学习。