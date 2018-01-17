# Associations 关联

这一节描述 sequelize 中不同的关联类型。在调用一个类似 `User.hasOne(Project)` 的方法时，我们把模型 `User`（正在调用函数的模型）叫做 __源__，把模型 `Project`（被当作参数传递的模型）叫做__目标__。

## One-To-One associations 一对一关联

一对一关联是通过单个外键连接的两个模型之间的关联。

### BelongsTo

BelongsTo 关联是一对一关系的外键存在于**源模型**上的关联。
一个简单的例子是外键在 player 上，**Player** 是 **Team** 的一部分。

```js
const Player = this.sequelize.define('player', {/* attributes */});
const Team  = this.sequelize.define('team', {/* attributes */});

Player.belongsTo(Team); // 会向 Player 添加一个 teamId 属性以保持与 Team 的主键一致。
```

#### Foreign keys 外键

默认地，belongsTo 关系的外键由目标模型名与其主键名构成的。

默认形式为 `camelCase`（驼峰式），但如果源模型设置了 `underscored: true` 外键将会是 `snake_case`（蛇形）。

```js
const User = this.sequelize.define('user', {/* attributes */})
const Company  = this.sequelize.define('company', {/* attributes */});

User.belongsTo(Company); // Will add companyId to user

const User = this.sequelize.define('user', {/* attributes */}, {underscored: true})
const Company  = this.sequelize.define('company', {
  uuid: {
    type: Sequelize.UUID,
    primaryKey: true
  }
});

User.belongsTo(Company); // Will add company_uuid to user
```

在 `as` 被定义的情况下， 他将被用作替换目标模型名。

```js
const User = this.sequelize.define('user', {/* attributes */})
const UserRole  = this.sequelize.define('userRole', {/* attributes */});

User.belongsTo(UserRole, {as: 'role'}); // Adds roleId to user rather than userRoleId
```

任何情况下默认的外键可以被 `foreignKey` 选项覆盖。
当外键选项被使用, Sequelize 会使用它 as-is:

```js
const User = this.sequelize.define('user', {/* attributes */})
const Company  = this.sequelize.define('company', {/* attributes */});

User.belongsTo(Company, {foreignKey: 'fk_company'}); // Adds fk_company to User
```

#### Target keys

目标模型上的目标键是源模型上外键指向的那一列。 belongsTo 关系的默认目标键是目标模型的主键，使用 `targetKey` 选项来定义一个自定义的列。

```js
const User = this.sequelize.define('user', {/* attributes */})
const Company  = this.sequelize.define('company', {/* attributes */});

User.belongsTo(Company, {foreignKey: 'fk_companyname', targetKey: 'name'}); // Adds fk_companyname to User
```


### HasOne

HasOne 关联是一对一关系的外键存在于**目标模型**上的关联。

```js
const User = sequelize.define('user', {/* ... */})
const Project = sequelize.define('project', {/* ... */})
 
// One-way associations
Project.hasOne(User)

/*
  在这个例子中 hasOne 添加一个 projectId 属性到 User 模型！
  此外， Project.prototype 根据传递给 define 的第一个参数获得 getUser and setUser 方法
  如果你启用了 underscore 样式
  被添加的属性将会以 project_id 替换 projectId。

  外键会被放置在 user 表中

  你也可以自己定义外键， 
  例如：如果你想在一个已经存在的数据库上工作 ：
*/
 
Project.hasOne(User, { foreignKey: 'initiator_id' })
 
/*
  因为 Sequelize 会为访问器方法使用模型的名字 (define 的第一个参数)
  可以向 hasOne 传递一个特殊的选项 ：
*/
 
Project.hasOne(User, { as: 'Initiator' })
// Now you will get Project#getInitiator and Project#setInitiator
 
// Or let's define some self references
const Person = sequelize.define('person', { /* ... */})
 
Person.hasOne(Person, {as: 'Father'})
// this will add the attribute FatherId to Person
 
// also possible:
Person.hasOne(Person, {as: 'Father', foreignKey: 'DadId'})
// this will add the attribute DadId to Person
 
// In both cases you will be able to do:
Person#setFather
Person#getFather
 
// If you need to join a table twice you can double join the same table
Team.hasOne(Game, {as: 'HomeTeam', foreignKey : 'homeTeamId'});
Team.hasOne(Game, {as: 'AwayTeam', foreignKey : 'awayTeamId'});

Game.belongsTo(Team);
```

尽管它被称作 HasOne 关联，对于大多数 1:1 关系你通常需要 BelongsTo 关联，因为 BelongsTo 在源上添加外键而 hasOne 在目标上添加外键。

### Difference between HasOne and BelongsTo

In Sequelize 1:1 relationship can be set using HasOne and BelongsTo. They are suitable for different scenarios. Lets study this difference using an example.

Suppose we have two tables to link **Player** and **Team**. Lets define their models.

```js
const Player = this.sequelize.define('player', {/* attributes */})
const Team  = this.sequelize.define('team', {/* attributes */});
```

When we link two models in Sequelize we can refer them as pairs of **source** and **target** models. Like this

Having **Player** as the **source** and **Team** as the **target**
```js
Player.belongsTo(Team);
//Or
Player.hasOne(Team);
```

Having **Team** as the **source** and **Player** as the **target**
```js
Team.belongsTo(Player);
//Or
Team.hasOne(Player);
```

HasOne and BelongsTo insert the association key in different models from each other. HasOne inserts the association key in **target** model whereas BelongsTo inserts the association key in the **source** model.

Here is an example demonstrating use cases of BelongsTo and HasOne.

```js
const Player = this.sequelize.define('player', {/* attributes */})
const Coach  = this.sequelize.define('coach', {/* attributes */})
const Team  = this.sequelize.define('team', {/* attributes */});
```

Suppose our `Player` model has information about its team as `teamId` column. Information about each Team's `Coach` is stored in the `Team` model as `coachId` column. These both scenarios requires different kind of 1:1 relation because foreign key relation is present on different models each time.

When information about association is present in **source** model we can use `belongsTo`. In this case `Player` is suitable for `belongsTo` because it has `teamId` column.

```js
Player.belongsTo(Team)  // `teamId` will be added on Player / Source model
```

When information about association is present in **target** model we can use `hasOne`. In this case `Coach` is suitable for `hasOne` because `Team` model store information about its `Coach` as `coachId` field.

```js
Coach.hasOne(Team)  // `coachId` will be added on Team / Target model
```

## One-To-Many associations

One-To-Many 关联是一个源连接多个目标。而目标只连接一个特定的源。
```js
const User = sequelize.define('user', {/* ... */})
const Project = sequelize.define('project', {/* ... */})
 
// OK. Now things get more complicated (not really visible to the user :)).
// First let's define a hasMany association
Project.hasMany(User, {as: 'Workers'})
```

This will add the attribute `projectId` or `project_id` to User. Instances of Project will get the accessors `getWorkers` and `setWorkers`. We could just leave it the way it is and let it be a one-way association.
But we want more! Let's define it the other way around by creating a many to many association in the next section:

Sometimes you may need to associate records on different columns, you may use `sourceKey` option:

```js
const City = sequelize.define('city', { countryCode: Sequelize.STRING });
const Country = sequelize.define('country', { isoCode: Sequelize.STRING });

// Here we can connect countries and cities base on country code
Country.hasMany(City, {foreignKey: 'countryCode', sourceKey: 'isoCode'});
City.belongsTo(Country, {foreignKey: 'countryCode', targetKey: 'isoCode'});
```


## Belongs-To-Many associations 多对多关联

Belongs-To-Many 关联用于一个源模型连接多个目标模型，同时一个目标模型也可以连接多个源模型。

```js
Project.belongsToMany(User, {through: 'UserProject'});
User.belongsToMany(Project, {through: 'UserProject'});
```

这将创建一个名为 UserProject 的新模型，with the equivalent foreign keys `projectId` and `userId`. 属性是否是驼峰式的取决于参与表格的两个模型（在这个例子中是 User 和 Project）。

定义 `through` 是**必要的**. Sequelize would previously attempt to autogenerate names but that would not always lead to the most logical setups.

这将为 `Project` 添加 `getUsers`, `setUsers`, `addUser`,`addUsers` 方法 , 同时为 `User` 添加 `getProjects`, `setProjects`, `addProject`, `addProjects` 方法

当你在联系中使用他们时，有时你可能想要重命名你的模型。 Let's define users as workers and projects as tasks by using the alias (`as`) option. 我们也可能会直接定义要是用的外键：
```js
User.belongsToMany(Project, { as: 'Tasks', through: 'worker_tasks', foreignKey: 'userId' })
Project.belongsToMany(User, { as: 'Workers', through: 'worker_tasks', foreignKey: 'projectId' })
```

`foreignKey` will allow you to set **source model** key in the **through** relation.
`otherKey` will allow you to set **target model** key in the **through** relation.

```js
User.belongsToMany(Project, { as: 'Tasks', through: 'worker_tasks', foreignKey: 'userId', otherKey: 'projectId'})
```

Of course you can also define self references with belongsToMany:

```js
Person.belongsToMany(Person, { as: 'Children', through: 'PersonChildren' })
// This will create the table PersonChildren which stores the ids of the objects.

```
If you want additional attributes in your join table, you can define a model for the join table in sequelize, before you define the association, and then tell sequelize that it should use that model for joining, instead of creating a new one:

```js
const User = sequelize.define('user', {})
const Project = sequelize.define('project', {})
const UserProjects = sequelize.define('userProjects', {
    status: DataTypes.STRING
})
 
User.belongsToMany(Project, { through: UserProjects })
Project.belongsToMany(User, { through: UserProjects })
```

To add a new project to a user and set its status, you pass extra `options.through` to the setter, which contains the attributes for the join table

```js
user.addProject(project, { through: { status: 'started' }})
```

By default the code above will add projectId and userId to the UserProjects table, and _remove any previously defined primary key attribute_ - the table will be uniquely identified by the combination of the keys of the two tables, and there is no reason to have other PK columns. To enforce a primary key on the `UserProjects` model you can add it manually.

```js
const UserProjects = sequelize.define('userProjects', {
  id: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  status: DataTypes.STRING
})
```
With Belongs-To-Many you can query based on **through** relation and select specific attributes. For example using `findAll` with **through**

```js
User.findAll({
  include: [{
    model: Project,
    through: {
      attributes: ['createdAt', 'startedAt', 'finishedAt'],
      where: {completed: true}
    }
  }]
});
```   

## Scopes
这一部分关注关联作用域。关于关联作用域和关联模型上的作用域定义的区别，参见 [Scopes](/manual/tutorial/scopes.html).

关联作用域允许你能够在关联上放置一个作用域（一套 `get` 和 `create` 的默认属性）。作用域既可以放置在关联模型上， (关联中的目标模型), 也可以放在新建 n:m 关系表格里。

#### 1:m
假设我们有 Comment（评论）, Post（帖子）, 和 Image（图片） 三张表。一条评论可以通过 `commentable_id` 和 `commentable` 属性与一张图片或帖子（其中一个）相关联 - 我们称 Post 和 Image 是 `Commentable` 的

```js
const Comment = this.sequelize.define('comment', {
  title: Sequelize.STRING,
  commentable: Sequelize.STRING,
  commentable_id: Sequelize.INTEGER
});

Comment.prototype.getItem = function(options) {
  return this['get' + this.get('commentable').substr(0, 1).toUpperCase() + this.get('commentable').substr(1)](options);
};

Post.hasMany(this.Comment, {
  foreignKey: 'commentable_id',
  constraints: false,
  scope: {
    commentable: 'post'
  }
});
Comment.belongsTo(this.Post, {
  foreignKey: 'commentable_id',
  constraints: false,
  as: 'post'
});

Image.hasMany(this.Comment, {
  foreignKey: 'commentable_id',
  constraints: false,
  scope: {
    commentable: 'image'
  }
});
Comment.belongsTo(this.Image, {
  foreignKey: 'commentable_id',
  constraints: false,
  as: 'image'
});
```

`constraints: false,` 禁用引用约束 - 因为 `commentable_id` 列引用多张表, 我们不能向它增加 `REFERENCES` 约束。注意 Image -> Comment 和 Post -> Comment 关系分别定义了 `commentable: 'image'` 和 `commentable: 'post'`。当使用关联函数时会自动应用作用域：

```js
image.getComments()
SELECT * FROM comments WHERE commentable_id = 42 AND commentable = 'image';

image.createComment({
  title: 'Awesome!'
})
INSERT INTO comments (title, commentable_id, commentable) VALUES ('Awesome!', 42, 'image');

image.addComment(comment);
UPDATE comments SET commentable_id = 42, commentable = 'image'
```

`Comment` 上的工具函数 `getItem` 完成这样的作用 - 它简单的将 `commentable` 上的字符串转换为 `getImage` 或 `getPost` 这样的一个调用 , 提供了一个评论属于帖子或图片的抽象。 你可以传递一个正常的设置对象作为 `getItem(options)` 的参数，来指定任何条件或者预载。

#### n:m
Continuing with the idea of a polymorphic model, consider a tag table - 一个 item 可以有多个 tag,，一个 tag 可以和多个 item 相关。

简化来看, 示例仅仅展示了一个 Post 模型, 但是实际上 Tag 会和其他的多个模型相关联。

```js
const ItemTag = sequelize.define('item_tag', {
  id : {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  tag_id: {
    type: DataTypes.INTEGER,
    unique: 'item_tag_taggable'
  },
  taggable: {
    type: DataTypes.STRING,
    unique: 'item_tag_taggable'
  },
  taggable_id: {
    type: DataTypes.INTEGER,
    unique: 'item_tag_taggable',
    references: null
  }
});
const Tag = sequelize.define('tag', {
  name: DataTypes.STRING
});

Post.belongsToMany(Tag, {
  through: {
    model: ItemTag,
    unique: false,
    scope: {
      taggable: 'post'
    }
  },
  foreignKey: 'taggable_id',
  constraints: false
});
Tag.belongsToMany(Post, {
  through: {
    model: ItemTag,
    unique: false
  },
  foreignKey: 'tag_id',
  constraints: false
});
```

注意作用域列 (`taggable`) 现在在关系模型 (`ItemTag`) 上。

We could also define a more restrictive association, for example, to get all pending tags for a post by applying a scope of both the through model (`ItemTag`) and the target model (`Tag`):

```js
Post.hasMany(Tag, {
  through: {
    model: ItemTag,
    unique: false,
    scope: {
      taggable: 'post'
    }
  },
  scope: {
    status: 'pending'
  },
  as: 'pendingTags',
  foreignKey: 'taggable_id',
  constraints: false
});

Post.getPendingTags();
```
```sql
SELECT `tag`.*  INNER JOIN `item_tags` AS `item_tag`
ON `tag`.`id` = `item_tag`.`tagId`
  AND `item_tag`.`taggable_id` = 42
  AND `item_tag`.`taggable` = 'post'
WHERE (`tag`.`status` = 'pending');
```

`constraints: false` disables references constraints on the `taggable_id` column. Because the column is polymorphic, we cannot say that it `REFERENCES` a specific table.

## Naming strategy

默认地， sequelize会使用模型名称 (传递给 `sequelize.define` 的 name) 推断关联中使用模型的名称。举个例子，一个名称为 `user`的模型 will add the functions `get/set/add User` to instances of the associated model, and a property named `.user` in eager loading, while a model named `User` will add the same functions, but a property named `.User` (notice the upper case U) in eager loading.

As we've already seen, you can alias models in associations using `as`. In single associations (has one and belongs to), the alias should be singular, while for many associations (has many) it should be plural. Sequelize then uses the [inflection ][0]library to convert the alias to its singular form. However, this might not always work for irregular or non-english words. In this case, you can provide both the plural and the singular form of the alias:

```js
User.belongsToMany(Project, { as: { singular: 'task', plural: 'tasks' }})
// Notice that inflection has no problem singularizing tasks, this is just for illustrative purposes.
```

If you know that a model will always use the same alias in associations, you can provide it when creating the model

```js
const Project = sequelize.define('project', attributes, {
  name: {
    singular: 'task',
    plural: 'tasks',
  }
})
 
User.belongsToMany(Project);
```

This will add the functions `add/set/get Tasks` to user instances.

Remember, that using `as` to change the name of the association will also change the name of the foreign key. When using `as`, it is safest to also specify the foreign key.

```js
Invoice.belongsTo(Subscription)
Subscription.hasMany(Invoice)
```

Without `as`, this adds `subscriptionId` as expected. However, if you were to say `Invoice.belongsTo(Subscription, { as: 'TheSubscription' })`, you will have both `subscriptionId` and `theSubscriptionId`, because sequelize is not smart enough to figure that the calls are two sides of the same relation. 'foreignKey' fixes this problem;

```js
Invoice.belongsTo(Subscription, , { as: 'TheSubscription', foreignKey: 'subscription_id' })
Subscription.hasMany(Invoice, { foreignKey: 'subscription_id' )
```

## Associating objects

Because Sequelize is doing a lot of magic, you have to call `Sequelize.sync` after setting the associations! Doing so will allow you the following:

```js
Project.belongsToMany(Task)
Task.belongsToMany(Project)
 
Project.create()...
Task.create()...
Task.create()...
 
// save them... and then:
project.setTasks([task1, task2]).then(() => {
  // saved!
})
 
// ok, now they are saved... how do I get them later on?
project.getTasks().then(associatedTasks => {
  // associatedTasks is an array of tasks
})
 
// You can also pass filters to the getter method.
// They are equal to the options you can pass to a usual finder method.
project.getTasks({ where: 'id > 10' }).then(tasks => {
  // tasks with an id greater than 10 :)
})
 
// You can also only retrieve certain fields of a associated object.
project.getTasks({attributes: ['title']}).then(tasks => {
  // retrieve tasks with the attributes "title" and "id"
})
```

To remove created associations you can just call the set method without a specific id:

```js
// remove the association with task1
project.setTasks([task2]).then(associatedTasks => {
  // you will get task2 only
})
 
// remove 'em all
project.setTasks([]).then(associatedTasks => {
  // you will get an empty array
})
 
// or remove 'em more directly
project.removeTask(task1).then(() => {
  // it's gone
})
 
// and add 'em again
project.addTask(task1).then(function() {
  // it's back again
})
```

You can of course also do it vice versa:

```js
// project is associated with task1 and task2
task2.setProject(null).then(function() {
  // and it's gone
})
```

For hasOne/belongsTo it's basically the same:

```js
Task.hasOne(User, {as: "Author"})
Task#setAuthor(anAuthor)
```

Adding associations to a relation with a custom join table can be done in two ways (continuing with the associations defined in the previous chapter):

```js
// Either by adding a property with the name of the join table model to the object, before creating the association
project.UserProjects = {
  status: 'active'
}
u.addProject(project)
 
// Or by providing a second options.through argument when adding the association, containing the data that should go in the join table
u.addProject(project, { through: { status: 'active' }})
 
 
// When associating multiple objects, you can combine the two options above. In this case the second argument
// will be treated as a defaults object, that will be used if no data is provided
project1.UserProjects = {
    status: 'inactive'
}
 
u.setProjects([project1, project2], { through: { status: 'active' }})
// The code above will record inactive for project one, and active for project two in the join table
```

When getting data on an association that has a custom join table, the data from the join table will be returned as a DAO instance:

```js
u.getProjects().then(projects => {
  const project = projects[0]
 
  if (project.UserProjects.status === 'active') {
    // .. do magic
 
    // since this is a real DAO instance, you can save it directly after you are done doing magic
    return project.UserProjects.save()
  }
})
```

If you only need some of the attributes from the join table, you can provide an array with the attributes you want:

```js
// This will select only name from the Projects table, and only status from the UserProjects table
user.getProjects({ attributes: ['name'], joinTableAttributes: ['status']})
```

## Check associations
You can also check if an object is already associated with another one (N:M only). Here is how you'd do it:

```js
// check if an object is one of associated ones:
Project.create({ /* */ }).then(project => {
  return User.create({ /* */ }).then(user => {
    return project.hasUser(user).then(result => {
      // result would be false
      return project.addUser(user).then(() => {
        return project.hasUser(user).then(result => {
          // result would be true
        })
      })
    })
  })
})
 
// check if all associated objects are as expected:
// let's assume we have already a project and two users
project.setUsers([user1, user2]).then(() => {
  return project.hasUsers([user1]);
}).then(result => {
  // result would be false
  return project.hasUsers([user1, user2]);
}).then(result => {
  // result would be true
})
```

## Foreign Keys

When you create associations between your models in sequelize, foreign key references with constraints will automatically be created. The setup below:

```js
const Task = this.sequelize.define('task', { title: Sequelize.STRING })
const User = this.sequelize.define('user', { username: Sequelize.STRING })
 
User.hasMany(Task)
Task.belongsTo(User)
```

Will generate the following SQL:

```sql
CREATE TABLE IF NOT EXISTS `User` (
  `id` INTEGER PRIMARY KEY,
  `username` VARCHAR(255)
);

CREATE TABLE IF NOT EXISTS `Task` (
  `id` INTEGER PRIMARY KEY,
  `title` VARCHAR(255),
  `user_id` INTEGER REFERENCES `User` (`id`) ON DELETE SET NULL ON UPDATE CASCADE
);
```

The relation between task and user injects the `user_id` foreign key on tasks, and marks it as a reference to the `User` table. By default `user_id` will be set to `NULL` if the referenced user is deleted, and updated if the id of the user id updated. These options can be overridden by passing `onUpdate` and `onDelete` options to the association calls. The validation options are `RESTRICT, CASCADE, NO ACTION, SET DEFAULT, SET NULL`.

For 1:1 and 1:m associations the default option is `SET NULL` for deletion, and `CASCADE` for updates. For n:m, the default for both is `CASCADE`. This means, that if you delete or update a row from one side of an n:m association, all the rows in the join table referencing that row will also be deleted or updated.

Adding constraints between tables means that tables must be created in the database in a certain order, when using `sequelize.sync`. If Task has a reference to User, the User table must be created before the Task table can be created. This can sometimes lead to circular references, where sequelize cannot find an order in which to sync. Imagine a scenario of documents and versions. A document can have multiple versions, and for convenience, a document has a reference to its current version.

```js
const Document = this.sequelize.define('document', {
  author: Sequelize.STRING
})
const Version = this.sequelize.define('version', {
  timestamp: Sequelize.DATE
})

Document.hasMany(Version) // This adds document_id to version
Document.belongsTo(Version, { as: 'Current', foreignKey: 'current_version_id'}) // This adds current_version_id to document
```

However, the code above will result in the following error: `Cyclic dependency found. 'Document' is dependent of itself. Dependency Chain: Document -> Version => Document`. In order to alleviate that, we can pass `constraints: false` to one of the associations:

```js
Document.hasMany(Version)
Document.belongsTo(Version, { as: 'Current', foreignKey: 'current_version_id', constraints: false})
```

Which will allow us to sync the tables correctly:

```sql
CREATE TABLE IF NOT EXISTS `Document` (
  `id` INTEGER PRIMARY KEY,
  `author` VARCHAR(255),
  `current_version_id` INTEGER
);
CREATE TABLE IF NOT EXISTS `Version` (
  `id` INTEGER PRIMARY KEY,
  `timestamp` DATETIME,
  `document_id` INTEGER REFERENCES `Document` (`id`) ON DELETE SET NULL ON UPDATE CASCADE
);
```

### Enforcing a foreign key reference without constraints

Sometimes you may want to reference another table, without adding any constraints, or associations. In that case you can manually add the reference attributes to your schema definition, and mark the relations between them.

```js 
// Series has a trainer_id=Trainer.id foreign reference key after we call Trainer.hasMany(series)
const Series = sequelize.define('series', {
  title:        DataTypes.STRING,
  sub_title:    DataTypes.STRING,
  description:  DataTypes.TEXT,
 
  // Set FK relationship (hasMany) with `Trainer`
  trainer_id: {
    type: DataTypes.INTEGER,
    references: {
      model: "trainers",
      key: "id"
    }
  }
})
 
const Trainer = sequelize.define('trainer', {
  first_name: DataTypes.STRING,
  last_name:  DataTypes.STRING
});
 
// Video has a series_id=Series.id foreign reference key after we call Series.hasOne(Video)...
const Video = sequelize.define('video', {
  title:        DataTypes.STRING,
  sequence:     DataTypes.INTEGER,
  description:  DataTypes.TEXT,
 
  // set relationship (hasOne) with `Series`
  series_id: {
    type: DataTypes.INTEGER,
    references: {
      model: Series, // Can be both a string representing the table name, or a reference to the model
      key:   "id"
    }
  }
});
 
Series.hasOne(Video);
Trainer.hasMany(Series);
```

## Creating with associations

An instance can be created with nested association in one step, provided all elements are new.

### Creating elements of a "BelongsTo", "Has Many" or "HasOne" association

Consider the following models:

```js
const Product = this.sequelize.define('product', {
  title: Sequelize.STRING
});
const User = this.sequelize.define('user', {
  first_name: Sequelize.STRING,
  last_name: Sequelize.STRING
});
const Address = this.sequelize.define('address', {
  type: Sequelize.STRING,
  line_1: Sequelize.STRING,
  line_2: Sequelize.STRING,
  city: Sequelize.STRING,
  state: Sequelize.STRING,
  zip: Sequelize.STRING,
});

Product.User = Product.belongsTo(User);
User.Addresses = User.hasMany(Address);
// Also works for `hasOne`
```

A new `Product`, `User`, and one or more `Address` can be created in one step in the following way:

```js
return Product.create({
  title: 'Chair',
  user: {
    first_name: 'Mick',
    last_name: 'Broadstone',
    addresses: [{
      type: 'home',
      line_1: '100 Main St.',
      city: 'Austin',
      state: 'TX',
      zip: '78704'
    }]
  }
}, {
  include: [{
    association: Product.User,
    include: [ User.Addresses ]
  }]
});
```

Here, our user model is called `user`, with a lowercase u - This means that the property in the object should also be `user`. If the name given to `sequelize.define` was `User`, the key in the object should also be `User`. Likewise for `addresses`, except it's pluralized being a `hasMany` association.

### Creating elements of a "BelongsTo" association with an alias

The previous example can be extended to support an association alias.

```js
const Creator = Product.belongsTo(User, {as: 'creator'});

return Product.create({
  title: 'Chair',
  creator: {
    first_name: 'Matt',
    last_name: 'Hansen'
  }
}, {
  include: [ Creator ]
});
```

### Creating elements of a "HasMany" or "BelongsToMany" association

Let's introduce the ability to associate a product with many tags. Setting up the models could look like:

```js
const Tag = this.sequelize.define('tag', {
  name: Sequelize.STRING
});

Product.hasMany(Tag);
// Also works for `belongsToMany`.
```

Now we can create a product with multiple tags in the following way:

```js
Product.create({
  id: 1,
  title: 'Chair',
  tags: [
    { name: 'Alpha'},
    { name: 'Beta'}
  ]
}, {
  include: [ Tag ]
})
```

And, we can modify this example to support an alias as well:

```js
const Categories = Product.hasMany(Tag, {as: 'categories'});

Product.create({
  id: 1,
  title: 'Chair',
  categories: [
    {id: 1, name: 'Alpha'},
    {id: 2, name: 'Beta'}
  ]
}, {
  include: [{
    model: Categories,
    as: 'categories'
  }]
})
```

***

[0]: https://www.npmjs.org/package/inflection
