Title: Extjs的十一种布局样式
Date: 2013-05-08

[参考文档:](http://dev.sencha.com/deploy/ext-4.0.0/examples/layout-browser/layout-browser.html)

### Absolute
类名: Ext.layout.container.Absolute
描述：这是最简单的布局风格，设置x,y坐标来确定绝对位置。
代码：
```
layout: 'absolute',
items:[{
    title: 'Panel 1',
    x: 50,
    y: 50,
    html: 'Positioned at x:50, y:50'
}]
```

### Accordion
类名：Ext.layout.container.Accordion
描述：手风琴效果，可折叠式的布局
代码：
```
layout: 'accordion',
items:[{
    title: 'Panel 1',
    html: 'Content'
},{
    title: 'Panel 2',
    id: 'panel2',
    html: 'Content'
}]
```

### Anchor
类名：Ext.layout.container.Anchor
描述：Anchor固定，这种布局常用于设置子控件相对于容器的位置。在anchor中指定高/宽,anchor通常为负值（不包含百分比）。
代码：
```
layout: 'anchor',
items: [{
    title: 'Panel 1',
    height: 100,
    anchor: '50%'
},{
    title: 'Panel 2',
    height: 100,
    anchor: '-100'
},{
    title: 'Panel 3',
    anchor: '-10, -262'
}]
```

### Border
类名：Ext.layout.container.Border
描述：将容器分为五个区域：east,south,west,north,center
代码：
```
layout:'border',
defaults: {
    collapsible: true,
    split: true,
    bodyStyle: 'padding:15px'
},
items: [{
    title: 'Footer',
    region: 'south',
    height: 150,
    minSize: 75,
    maxSize: 250,
    cmargins: '5 0 0 0'
},{
    title: 'Navigation',
    region:'west',
    margins: '5 0 0 0',
    cmargins: '5 5 0 0',
    width: 175,
    minSize: 100,
    maxSize: 250
},{
    title: 'Main Content',
    collapsible: false,
    region:'center',
    margins: '5 0 0 0'
}]
```

### TabPanel
类名：Ext.layout.container.Card (TabPanel)
描述：标签页布局，一次展现一个标签内容，在使用的时候不要忘记设置一项为默认展现的标签页
代码：
```
xtype: 'tabpanel',
activeTab: 0, // index or id
items:[{
    title: 'Tab 1',
    html: 'This is tab 1 content.'
},{
    title: 'Tab 2',
    html: 'This is tab 2 content.'
},{
    title: 'Tab 3',
    html: 'This is tab 3 content.'
}]
```

### Wizard
类名：Ext.layout.container.Card (Wizard)
描述：布局名称card，用于创建普通的wizard风格的界面，同时开发者必须编写导航的函数实现wizard's的业务逻辑
代码：
```
layout:'card',
activeItem: 0, // index or id
bbar: ['->', {
    id: 'card-prev',
    text: '&laquo; Previous'
},{
    id: 'card-next',
    text: 'Next &raquo;'
}],
items: [{
    id: 'card-0',
    html: 'Step 1'
},{
    id: 'card-1',
    html: 'Step 2'
},{
    id: 'card-2',
    html: 'Step 3'
}]
```

### Column
类名：Ext.layout.container.Column
描述：列布局方式，通过设定容器的宽度将多个容器放置在同一行，设定的方式包括精确的数值和百分比。
代码：
```
layout:'column',
items: [{
    title: 'Width = 25%',
    columnWidth: .25,
    html: 'Content'
},{
    title: 'Width = 75%',
    columnWidth: .75,
    html: 'Content'
},{
    title: 'Width = 250px',
    width: 250,
    html: 'Content'
}]
```

### Fit
类名：Ext.layout.container.Fit
描述：子容器充满父容器（如果有多个就只有一个充满容器）
代码：
```
layout:'fit',
items: {
    title: 'Fit Panel',
    html: 'Content',
    border: false
}
```

### Table
类名：Ext.layout.container.Table
描述：类似于HTML表格的样式
代码：
```
layout: {
    type: 'table',
    columns: 4
},
items: [
    {html:'1,1',rowspan:3},
    {html:'1,2'},
    {html:'1,3'},
    {html:'2,2',colspan:2},
    {html:'3,2'},
    {html:'3,3'}
]
```

### vBox
类名：Ext.layout.container.VBox
描述：垂直布局
代码：
```
layout: {
    type: 'vbox'
    align : 'stretch',
    pack  : 'start',
},
items: [
    {html:'panel 1', flex:1},
    {html:'panel 2', height:150},
    {html:'panel 3', flex:2}
]
```

### hBox
类名：Ext.layout.container.HBox
描述：水平布局
代码：
```
layout: {
    type: 'hbox',
    pack: 'start',
    align: 'stretch'
},
items: [
    {html:'panel 1', flex:1},
    {html:'panel 2', width:150},
    {html:'panel 3', flex:2}
]
```
