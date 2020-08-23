# Node Web开发ORM框架 Sequelize

### 一、什么是 ORM？

首先看下维基百科上的定义，ORM 是「对象关系映射」的翻译，英语全称为Object Relational Mapping，它是一种程序设计技术，用于实现面向对象编程语言里不同类型系统的数据之间的转换。从效果上说，它其实是创建了一个可在编程语言里使用的「虚拟对象数据库」。

随着面向对象软件开发方法的发展，ORM 的概念应运而生，它用来把对象模型表示的对象，映射到基于 SQL 的关系模型数据库结构中去。这样，我们在具体的操作实体数据库的时候，就不需要再去和复杂的 SQL 语句打交道，只需简单的操作实体对象的属性和方法，就可以达到操作数据库的效果。

ORM 技术是在对象和数据库之间提供了一条桥梁，前台的对象型数据和数据库中的关系型的数据通过这个桥梁来相互转化。

不同的编程语言，有不同的ORM框架。例如Java，它的ORM框架就有：Hibernate，Ibatis/Mybatis等等。在Node Web开发中，[Sequelize](http://docs.sequelizejs.com/manual/installation/getting-started.html) 就是一款比较流行的 ORM 框架。

### 二、Sequelize 初步使用

在 Node Web 开发过程中，后台数据库我一直使用的都是 Mysql。起初在做 Node Web 开发的时候，都是提前在 Mysql 图形界面里创建好数据表，然后再开始实际开发，这个过程一直穿插在整个项目的开发过程中。一个人在一台机器上，做全栈的开发，这个过程可能并不会出现什么问题，因为数据表结构以及整个项目代码都在一台电脑上，不管你怎么修改，都是一套代码，一个数据库结构。

然而，当你需要在多台电脑之间协同工作的时候，你就会发现这种方式的弊端。比如在A电脑上修改了数据表结构之后，接着去B电脑上继续编码，我们虽然能通过Git同步代码，但是数据表结构却无法同步过去，我们就需要在B电脑上，手动将数据库结构维护成一致，否则无法接着进行。这种操作方式非常的不方便，而且很LOW。

而 Sequelize 框架就能很好的解决这个问题，通过 Sequelize 框架，我们将每个数据表直接定义为数据模型，通过调用数据模型的一些方法，就可以直接操作数据库，甚至是同步数据表结构。好了，废话不多说了，直接上手实战。

> PS. 下面我会介绍，我在实际开发中的一些实用技巧，而不会逐条介绍增删改查的使用。所以增删改查的基本使用，建议直接查看[官方文档](http://docs.sequelizejs.com/)。

#### 1. 创建连接对象，并模块化

新建数据库连接模块`dbConn.js`，单独提出连接数据库的对象`sequelize`，如下代码：

```javascript
var Sequelize = require('sequelize');
// 数据库配置文件
var sqlConfig = {
    host: "localhost",
    user: "root",
    password: "Lupeng1",
    database: "example-sequelize"
};

var sequelize = new Sequelize(sqlConfig.database, sqlConfig.user, sqlConfig.password, {
    host: sqlConfig.host,
    dialect: 'mysql',
    pool: {
        max: 10,
        min: 0,
        idle: 10000
    }
});
module.exports = sequelize;
```

我们根据数据库的一些参数，创建了`sequelize`数据库连接模块，并对外引用。

#### 2. 定义数据表结构，将表结构写进代码里

建议每个表对应一个文档，放入项目的单独目录下，例如我通常放进了`/db/models`下，这里拿我创建的一个`todolist`表来做示例，在`models`目录中创建`todolist.js`文件，代码如下：

```javascript
var Sequelize = require('sequelize');
var sequelize = require('./dbConn.js');

var todolist = sequelize.define('todolist',{
    id: {
        type: Sequelize.BIGINT(11),
        primaryKey: true,
        allowNull: false,
        unique: true,
        autoIncrement: true
    },
    title: Sequelize.STRING(100),          // 标题
    content: Sequelize.STRING(500),        // 详细内容
    priority: Sequelize.INTEGER,          // 级别
    owner: Sequelize.STRING,              // 承接人
    officer: Sequelize.STRING,             // 负责人
    startDate: Sequelize.STRING,         // 开始时间
    planFinishDate: Sequelize.STRING,     // 计划完成时间
    realFinishDate: Sequelize.STRING,     // 实际完成时间
    bz: Sequelize.STRING(500),               // 备注
    state: Sequelize.INTEGER,            // 状态
    createdAt: Sequelize.BIGINT,
    updatedAt: Sequelize.BIGINT,
    createUser: Sequelize.STRING,
    updateUser: Sequelize.STRING,
    version: Sequelize.BIGINT
},{
    timestamps: false               // 不要默认时间戳
});

module.exports = todolist;
```

以上代码，直接引入之前创建的`sequelize`对象，然后使用`defind`方法定义数据表结构。其他的所有数据表都可以通过这种方式来定义，保存在每一个独立的文件中，引出数据模模型即可。

#### 3. 同步数据表结构

这个就简单了，在`/db`目录下，新建`syncTable.js`，代码如下：

```javascript
var todolist = require('./models/todolist.js');

// 同步表结构
todolist.sync({
    force: true  // 强制同步，先删除表，然后新建
});
```

当我们换台电脑继续项目的时候，不用手动去同步数据表结构了，只需要执行一下该文件就可以了。

```undefined
node db/syncTable.js
```

#### 4. 创建一些初始数据

我们同样可以创建一个方法，用来初始化一些基础数据，如下`initData.js`代码：

```javascript
var priority = require('./models/priority.js');
// 创建u_priority表的基础数据
priority.create({
    title: '重要 紧急'
}).then(function (p) {
    console.log('created. ' + JSON.stringify(p));
}).catch(function (err) {
    console.log('failed: ' + err);
});
...
```

给`prioritys`表里创建一些初始数据，默认表名会添加s，定义表的时候可以通过`tableName`属性值来定义对应的表名，如下示例将表名定义为`u_priority`：

```javascript
var priority = sequelize.define('priority',{
    id: {
        type: Sequelize.BIGINT(11),
        primaryKey: true,
        allowNull: false,
        autoIncrement: true
    },
    title: Sequelize.STRING,
},{
    timestamps: false,
    tableName: 'u_priority'  // 数据表名为u_priority
});
```

`Sequelize` 的初步使用，就介绍到这里。接下来，通过实际项目示例，再深入了解一下 `Sequelize` 的其他功效。

### 三、实际项目示例

#### 1. 协同开发规范

实际项目中，直接面临的一个问题就是**协同规范**的问题，例如，数据表名命名的规则，是采用默认方案，直接在后面加s，变成`prioritys`，还是改成`u_priority`；还有，每个表是否要加时间戳，或者其他一些通用字段。

如果没有规范，那么同一个项目中，数据表的形式就会百花齐放了，这是个人开发习惯问题，并不是错误。所以，当在做协同开发的时候，我们非常有必要定义一个标准接口，大家通过统一的标准调用方法建立模型，这样就形成一种内部规范。我们把所有需要规范的内容，全部封装在一个对象里，例如我们这样改写`dbConn.js`文件。

```tsx
var Sequelize = require('sequelize');
// 数据库配置文件
var sqlConfig = {
    host: "localhost",
    user: "root",
    password: "Lupeng1",
    database: "example-sequelize"
};

console.log('init sequelize...');
console.log('mysql: ' + JSON.stringify(sqlConfig));

var sequelize = new Sequelize(sqlConfig.database, sqlConfig.user, sqlConfig.password, {
    host: sqlConfig.host,
    dialect: 'mysql',
    pool: {
        max: 10,
        min: 0,
        idle: 10000
    },
    timezone: '+08:00' //东八时区
});

exports.sequelize = sequelize;

exports.defineModel = function (name, attributes) {
    var attrs = {};
    for (let key in attributes) {
        let value = attributes[key];
        if (typeof value === 'object' && value['type']) {
            value.allowNull = value.allowNull || false;
            attrs[key] = value;
        } else {
            attrs[key] = {
                type: value,
                // allowNull: false
            };
        }
    }
    attrs.version = {
        type: Sequelize.BIGINT,
        // allowNull: false
    };
    attrs.createUser = {
        type: Sequelize.STRING,
        allowNull: false
    };
    attrs.updateUser = {
        type: Sequelize.STRING,
        allowNull: false
    };
    return sequelize.define(name, attrs, {
        tableName: name,
        timestamps: true,
        paranoid: true, 
        charset: 'utf8mb4', 
        collate: 'utf8mb4_general_ci',
        hooks: {
            beforeBulkCreate: function(obj){
                obj.version = 0 ;
            },
            beforeValidate: function(obj){
                if(obj.isNewRecord){
                    console.log('first');
                    obj.version = 0 ; 
                }else{
                    console.log('not first');
                    obj.version = obj.version + 1 ;
                }
            }
        }
    });
};
```

我们在`dbConn.js`中，定义了`defineModel`方法，这个方法中，我们规范了，每个模型都要添加`version`、`createUser`以及`updateUser`三个字段、表名即为模型名，以及数据的字符集为`utf8mb4`等等。那么，当我们要创建模型的时候，通过调用`defineModel`方法就可以达到统一规范的效果。如下模型代码：

```tsx
var Sequelize = require('sequelize');
var db = require('../dbConn.js');

module.exports = db.defineModel('project_master', {
    p_id: {
        type: Sequelize.BIGINT(11),
        primaryKey: true,
        allowNull: false,
        autoIncrement: true
    },
    p_name: Sequelize.STRING(100),
    p_academy: Sequelize.STRING(100),
    p_start_date: Sequelize.STRING(10),
    p_end_date: Sequelize.STRING(10),
    p_days: Sequelize.DECIMAL(10, 1),
    p_place: Sequelize.STRING(20),
    p_owner: Sequelize.STRING(10),
    p_operator: Sequelize.STRING(10),
    p_is_fee: Sequelize.BIGINT(1),
    p_state: Sequelize.BIGINT(2),  // 开启，关闭
    p_bz: Sequelize.STRING(255),
});
```

这样还有一个好处，那就是在模型定义里，只需要写业务相关的字段，与业务无关的一些通用字段（例如，时间戳等等）就全部放到规范里。业务字段一目了然，显得更加清晰。

#### 2. 批量同步数据结构

上面介绍过，在同步数据结构的时候，我们可以通过`sync`方法进行同步，这里介绍一种批量同步的方法。正常情况下，我们都会把数据模型放在一个目录下`models`，于是我们在`models`目录外，新建一个`sync.js`文件。

```js
var sequelize = require('./dbConn.js').sequelize;
var fs = require('fs');
var files = fs.readdirSync(__dirname + '/models');
var js_files = files.filter((f)=>{
    return f.endsWith('.js');
}, files);
console.log(js_files);
module.exports = {};
for (var f of js_files) {
    console.log(`import model from file ${f}...`);
    var name = f.substring(0, f.length - 3);
    module.exports[name] = require(__dirname + '/models/' + f);
}
sequelize.sync();
```

该方法的本质其实就是自动找出`models`目录下，所有以`js`结尾的文件，引入并执行`sync()`方法。命令行执行`node db/sync.js`后，效果如下图：

![img](https:////upload-images.jianshu.io/upload_images/128602-851cf5db1033aaf6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

sync

再查看数据库情况，可以发现`models`目录下的所有模型全部创建成功。

![img](https:////upload-images.jianshu.io/upload_images/128602-e9e1864418da4266.png?imageMogr2/auto-orient/strip|imageView2/2/w/420/format/webp)

image.png

#### 3. 数据表关系结构

我们知道数据表的关系有**一对一**，**一对多**以及**多对多**的关系结构。一般 ORM 框架都会提供与之对应的对象方法，当然 Sequelize 也不例外。实战项目，直接上代码说话。新建一个`relation.js` 文件，用来定义模型之间的关系，代码如下：

```tsx
// 项目表
var ProjectMaster = require('./models/Project-master');
var ProjectCost = require('./models/Project-cost');
var ProjectState = require('./models/Project-state');
// 费用明细表
var TeachFee = require('./models/Detail-teach-fee.js');

ProjectMaster.hasOne(ProjectState, {foreignKey: 'p_id', as: 'State'});
ProjectMaster.hasOne(ProjectCost, {foreignKey: 'p_id', as: 'Cost'});
ProjectMaster.hasMany(TeachFee, {foreignKey: 'p_id', as: 'TeachFee'});

module.exports = {
    ProjectMaster: ProjectMaster,
    ProjectCost: ProjectCost,
    ProjectState: ProjectState,
    TeachFee: TeachFee
};
```

这里介绍一对一，一对多的关系，示例中，`ProjectMaster`与`ProjectCost`, `ProjectState`是一对一的关系，于是我们使用`hasOne`的方法，并且指定字段`p_id`为连接外键。而`TeachFee`是费用明细表，与项目表是一对多的关系，于是使用`hasMany`的方法，同样指定外键。

当定义完数据表关系后，我们重新编辑同步数据表方法，因为我们需要使用新定义的关系模型，新建`sync2.js`文件，代码如下：

```jsx
var sequelize = require('./dbConn.js').sequelize;
var relation = require('./relation.js');

module.exports = {
  ProjectMaster: relation.ProjectMaster,
  ProjectCost: relation.ProjectCost,
  ProjectState: relation.ProjectState,
  TeachFee: relation.TeachFee
};

sequelize.sync({
    force: true      // 强制同步
});
```

执行`node db/sync2.js`后，在表结构中，你会发现出现的外键关联关系。上述代码会在`ProjectMaster`、`ProjectCost`以及`TeachFee`模型对应的数据表中，新增了外键关联。如下状态表的情况。

```php
CREATE TABLE `project_state` (
  `p_state_id` bigint(11) NOT NULL AUTO_INCREMENT,
  `p_id` bigint(11) DEFAULT NULL,
  `p_state_teach` bigint(2) DEFAULT NULL,
  `p_state_place` bigint(2) DEFAULT NULL,
  `p_state_stay` bigint(2) DEFAULT NULL,
  `p_state_catering` bigint(2) DEFAULT NULL,
  `p_state_goods` bigint(2) DEFAULT NULL,
  `p_state_clean` bigint(2) DEFAULT NULL,
  `p_state_report` bigint(2) DEFAULT NULL,
  `version` bigint(20) DEFAULT NULL,
  `createUser` varchar(255) NOT NULL,
  `updateUser` varchar(255) NOT NULL,
  `createdAt` datetime NOT NULL,
  `updatedAt` datetime NOT NULL,
  `deletedAt` datetime DEFAULT NULL,
  PRIMARY KEY (`p_state_id`),
  KEY `p_id` (`p_id`),
  CONSTRAINT `project_state_ibfk_1` FOREIGN KEY (`p_id`) REFERENCES `project_master` (`p_id`) ON DELETE SET NULL ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

创建完表之后，接下来的问题就是我们如何使用？这里给个建议方案，我们在`db`目录下，新建一个`api`的目录，用来存放数据库调用接口。我们把所有对数据库的操作，全部写成方法，供后台路由调用。例如，对项目的查询，删除，修改等等操作，都定义成一个一个的方法。在`api`目录下，新建`projectModel.js`文件，用来定义对项目的一些数据库操作。代码如下：

```php
var sequelize = require('../dbConn.js').sequelize;

var ProjectMaster = require('../relation.js').ProjectMaster;
var ProjectCost = require('../relation.js').ProjectCost;
var ProjectState = require('../relation.js').ProjectState;

module.exports = {
    // 单表：仅更新项目表
    updateProject: function(data, id, callback){
        ProjectMaster.update(data ,{where: {p_id: id}}).then(function(p){
            callback();
        });
    }
    // 双表：查找成本表
    getCostList: function(start, end, callback){
        ProjectMaster.findAll({
            include: [{
                model: ProjectCost,
                as: 'Cost',
            }],
            where: {
                p_start_date: {
                    $lte: end,
                    $gte: start
                }
            },
            order: [sequelize.literal('p_start_date')]
        }).then(function(p){
            callback(p);
        });
    },
    // 双表：添加项目，同时在状态表添加项目状态
    addProject: function(data, callback){
        ProjectMaster.create(data).then(function(p){
            var state = ProjectState.build({
                p_state_teach: 1,
                p_state_stay: 1,
                p_state_catering: 1,
                p_state_place: 1,
                p_state_goods: 1,
                p_state_clean: 1,
                p_state_report: 1,
                createUser: data.createUser,
                updateUser: data.updateUser,
            });
            p.setState(state);
            callback(p);
        });
    },
};
```

由于对数据库的操作方法较多，这里用3个示例方法来介绍。

1. 第一个为单表操作，更新项目表。增删改查的一些基本方法，建议查看官方文档；
2. 第二个为双表联合查询，通过`include`参数来关联模型，得到的结果中会包含一个`Cost`对象，包含`ProjectCost`的模型数据；
3. 第三个为添加项目的时候，同时添加状态表，这里用到关系模型中的`set`方法，创建项目主数据表后，通过关联关系，使用`set`方法自动在State表中insert数据。

