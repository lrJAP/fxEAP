# lrEAP模式化开发

## 简述

lrEAP的界面模式，在JavaFX提供的基础组件之上，进行封装、增强、优化，针对我们在界面层面需要解决的问题，提供完整的UI模式。

常见的UI功能界面，主要有以下几个部分构成：

工具栏。提供一系列按钮，用于完成特定的操作。这些按钮，可能还与功能权限、数据权限、流程权限等相关。

业务数据展现及管理。以多种形式展现、管理业务数据。这部分是整个业务功能界面中最重要的内容。从业务数据的展现形式上看，有单表界面、主子表界面、单子多子表界面、多主多子表界面、树卡界面、树单表界面、树卡表界面、树卡单表界面、树卡多表界面等。从业务数据的管理功能上看，主要有两种形式实现业务数据的新增、修改、删除等功能。一是在Form界面中进行单条业务数据的管理。二是在Table界面中实现对单笔、多笔业务数据的管理。

## 界面结构

![UI](ui10.png)

## 界面组成

### MVC模式

![BillDesigner](mvc01.png)

lrEAP基于JavaFX的UI框架，参照了MVC模式进行设计。

Control层的主要体现，是一系列的Action，每一个Action，代表一个业务动作，在界面上以按钮的形式进行展现。

View层的主要体现，是一系列的View，采用不同的形式展现数据，如树（TreeView）、表格（TableView）、表单（FormView）。

Model层的主要体现，是各种模型，每一个模型，都是一个集成器，作为总线，沟通Control和View，为它们提供数据，处理、转发各类事件等。

1. View和Action之间不直接进行交互；

2. View和Action中，一般情况下，至少有一个AppModel；

3. View和Action均与AppModel进行交互；

4. View和Action均能够触发或响应AppModel定义、派发的事件。

5. Model负责进行远程调用，查询、提供数据，View、Action中只对这些数据进行处理。

### 通讯模式

![BillDesigner](mvc02.png)

AppModel在整个UI体系中，占有非常重要的地位，它处于幕后，不显示在界面上，不会直接与用户打交道。各类View和Action，负责界面的展现以及处理，直接面对用户。

从通讯模式上看，我们可以把这些组件分为五类：

第1类是AppModel。负责通过第4类辅助组件，获取数据、规范数据的展现形式；通过事件（Event）机制，把它周边所有的组件联系起来。

第2类是View。负责数据的展现。如表格、卡片、树等。

第3类是Action。负责对数据的操作。如查询、刷新、新增、修改、保存、删除等。

第4类是Manager、Proxy、Factory等类种辅助组件。它们的职责是协助AppModel进行工作。如调用后台业务处理类代码、查询单据模板、查询元数据等。

第5类是环境上下文。保存当前用户相关的信息、当前节点信息等。

例如，当我们在界面上点击“查询”按钮后，将由“查询”按钮对应的QueryAction向AppModel发送QueryEvent；AppModel按约定的规则，通过DataManager以及相关的辅助类进行数据查询操作；获取结果数据后，向所有注册了接收QueryCompletedEvent事件的2类组件发送消息；如果当前界面采用TableView展现数据，则此TableView在接收到事件后，将从AppModel中获取数据，并展现在界面上。

### 事件处理

![BillDesigner](mvc03.png)

lrEAP中的MVC模型之间的通讯，以AppModel为总线，派发、接收各类事件。Model、View、Action既是事件的产生者，又是事件是消费者，只是在产生、消费的事件类型上各有不同。所有的View、Action均需要作为事件接收者，在AppModel中进行注册，并在View、Action对监听的事件进行响应。也就是有如下约定：一个组件向AppModel声明需要监听事件的话，则应在其自身的代码中，响应该事件。

1. 在通讯层面上，每个Action均将触发一个或多个事件，同时接收特定的事件。例如AddAction，将触发数据新增事件，经由AppModel，派发给所有组件；同时AddAction接收界面状态事件，只有界面在非编辑状态时，AddAction才是有效的。

2. View中的数据与AppModel提供的数据进行双向绑定，任何一方的数据发生变化时，都会通过事件通知对方，例如通过DataManager查询并改变AppModel中的数据时，相应View中的数据会被通知进行变化；用户在TableView或FormView中改变数据后，AppModel中的数据也将即时更新。例如，当我们在TableView中选中数据，并点击“删除”时，将触发DeleteAction，AppModel接收到该事件后，将根据约定对其中的数据进行处理，完成后，向各组件派发“数据删除完成”事件。
3. AppModel的主要功能，就是作为集成、总线，接收事件、处理业务、派发事件。这些事件的操作对象，主要是保存在AppModel中的各种状态和数据。

### 容器结构

![BillDesigner](mvc04.png)

所有功能节点的根容器，都是一个BorderPane。在展现节点内容时，我们只使用它上、中、下三个区域，左、右区域不需要使用。

在Top区域中，将建立该节点中所有节点级的按钮，也就是工具栏区域。

在Center区域中，我们将显示单据模板（BillTemplate）。单据模板的根容器也是一个BorderPane，上、右、中、下、右五个区域均可以使用，根据业务需求及界面展现要求，确定需要使用的区域。

在Bottom区域中，显示该节点状态相关的信息。

## UIComponent

![BillDesigner](ui01.png)

### UIComponent-Container

![BillDesigner](ui02.png)

### UIComponent-Toolbar

![BillDesigner](ui03.png)

### UIComponent-ActionHGroup

![BillDesigner](ui05.png)

### UIComponent-Query

![BillDesigner](ui06.png)

### UIComponent-Action

![BillDesigner](ui07.png)

### UIComponent-View

![BillDesigner](ui08.png)

## AppModel

![BillDesigner](appmodel01.png)

## BillModel

![BillDesigner](billmodel01.png)

### Struct

![BillDesigner](billmodel02.png)

## 界面模式示例

### 单表-窗体（TableFormAppModel）

## 列表界面

![TableFormView](TableFormView03.png)

## 窗体界面

![TableFormView](TableFormView04.png)

### 批量编辑的单表（BatchTableAppModel）

## 列表界面

![BatchEditTableView](BatchEditTableView01.png)

### 带分页的表格-窗体（TableFormAppModel）

## 列表界面

![TableFormView](TableFormView01.png)

## 窗体界面

![TableFormView](TableFormView02.png)

### 主子表--子表为TableFormView

## 列表界面

![MasterDetailModelSubTableForm](MasterDetailModelSubTableForm01.png)

## 窗体界面

![MasterDetailModelSubTableForm](MasterDetailModelSubTableForm02.png)

### 主子表--子表为TableView

## 列表界面

![MasterDetailModelSubTable](MasterDetailModelSubTable01.png)

## 窗体界面

![MasterDetailModelSubTable](MasterDetailModelSubTable02.png)

### 树形结构（AppModel）

## 列表界面

![TreeModel](TreeModel01.png)

## 窗体界面

![TreeModel](TreeModel02.png)

### 树-窗体（TreeFormAppModel）

![TreeFormView](TreeFormView01.png)

### 树-表格-窗体（TreeTableFormAppModel）

## 列表界面

![TreePaginationTableView](TreePaginationTableView01.png)

## 窗体界面

![TreePaginationTableView](TreePaginationTableView02.png)

### 多对多结构（ManyToManyAppModel）

## 列表界面

![ManyToManyModel](ManyToManyModel01.png)

## 窗体界面

![ManyToManyModel](ManyToManyModel02.png)

### 树-树结构（AppModel）