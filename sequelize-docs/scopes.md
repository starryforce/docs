# Scopes
作用域允许你定义常用的查询语句以便之后使用。 Scopes can include all the same attributes as regular finders, `where`, `include`, `limit` etc.

## Definition

Scopes are defined in the model definition and can be finder objects, or functions returning finder objects - except for the default scope, which can only be an object:

```js
const Project = sequelize.define('project', {
  // Attributes
}, {
  defaultScope: {
    where: {
      active: true
    }
  },
  scopes: {
    deleted: {
      where: {
        deleted: true
      }
    },
    activeUsers: {
      include: [
        { model: User, where: { active: true }}
      ]
    },
    random: function () {
      return {
        where: {
          someNumber: Math.random()
        }
      }
    },
    accessLevel: function (value) {
      return {
        where: {
          accessLevel: {
            [Op.gte]: value
          }
        }
      }
    }
  }
});
```

定义模型后仍然可以通过调用 `addScope` 添加作用域。这对于具有 includes 语句的作用域特别有用， where the model in the include might not be defined at the time the other model is being defined.

默认作用域总是会被应用。这意味着，根据上面的模型定义，`Project.findAll()` 将创建下面的查询语句：

```sql
SELECT * FROM projects WHERE active = true
```

通过调用 `.unscoped()`，`.scope(null)`，或者调用其他作用域，默认作用域可以被移除。

```js
Project.scope('deleted').findAll(); // Removes the default scope
```
```sql
SELECT * FROM projects WHERE deleted = true
```

可以在一个作用域定义中嵌套已定义作用域的模型。This allows you to avoid duplicating `include`, `attributes` or `where` definitions.
Using the above example, and invoking the `active` scope on the included User model (rather than specifying the condition directly in that include object):

```js
activeUsers: {
  include: [
    { model: User.scope('active')}
  ]
}
```

## Usage
Scopes are applied by calling `.scope` on the model definition, passing the name of one or more scopes. `.scope` returns a fully functional model instance with all the regular methods: `.findAll`, `.update`, `.count`, `.destroy` etc. You can save this model instance and reuse it later:

```js
const DeletedProjects = Project.scope('deleted');

DeletedProjects.findAll();
// some time passes

// let's look for deleted projects again!
DeletedProjects.findAll();
```

Scopes apply to `.find`, `.findAll`, `.count`, `.update` and `.destroy`.

Scopes which are functions can be invoked in two ways. If the scope does not take any arguments it can be invoked as normally. If the scope takes arguments, pass an object:

```js
Project.scope('random', { method: ['accessLevel', 19]}).findAll();
```
```sql
SELECT * FROM projects WHERE someNumber = 42 AND accessLevel >= 19
```

## Merging
Several scopes can be applied simultaneously by passing an array of scopes to `.scope`, or by passing the scopes as consecutive arguments.

```js
// These two are equivalent
Project.scope('deleted', 'activeUsers').findAll();
Project.scope(['deleted', 'activeUsers']).findAll();
```
```sql
SELECT * FROM projects
INNER JOIN users ON projects.userId = users.id
AND users.active = true
```

If you want to apply another scope alongside the default scope, pass the key `defaultScope` to `.scope`:

```js
Project.scope('defaultScope', 'deleted').findAll();
```
```sql
SELECT * FROM projects WHERE active = true AND deleted = true
```

When invoking several scopes, keys from subsequent scopes will overwrite previous ones (similar to [_.assign](https://lodash.com/docs#assign)). Consider two scopes:

```js
{
  scope1: {
    where: {
      firstName: 'bob',
      age: {
        [Op.gt]: 20
      }
    },
    limit: 2
  },
  scope2: {
    where: {
      age: {
        [Op.gt]: 30
      }
    },
    limit: 10
  }
}
```

Calling `.scope('scope1', 'scope2')` will yield the following query

```sql
WHERE firstName = 'bob' AND age > 30 LIMIT 10
```

Note how `limit` and `age` are overwritten by `scope2`, while `firstName` is preserved. `limit`, `offset`, `order`, `paranoid`, `lock` and `raw` are overwritten, while `where` and `include` are shallowly merged. This means that identical keys in the where objects, and subsequent includes of the same model will both overwrite each other.

相同的合并逻辑 applies when passing a find object directly to findAll on a scoped model:

```js
Project.scope('deleted').findAll({
  where: {
    firstName: 'john'
  }
})
```
```sql
WHERE deleted = true AND firstName = 'john'
```

Here the `deleted` scope is merged with the finder. If we were to pass `where: { firstName: 'john', deleted: false }` to the finder, the `deleted` scope would be overwritten.

## Associations
Sequelize 中有两种与关联有关的作用域概念，它们不同但相关，之间的差异虽然细微但很重要：

* **关联作用域** 允许你在获取或设置关联时指定默认的属性（作用域内容） - 这在实现多对多关联时很有用。当使用 `get`, `set`, `add` 和 `create`等关联模型函数时，仅在两个模型的关联中调用这个这个作用域。
* **关联模型上的作用域** 允许你在获取关联时应用默认作用域或其他作用域，并且允许创建关联时传递一个有作用域的模型。这些作用域同时应用于模型上的常规查询，和通过关联的查询。

作为一个例子，考虑模型 Post（帖子） 和 Comment（评论）。Comment 和几种其他模型相关联（例如 Image, Video 等），并且 Comment 和 其他模型（Image, Video）的关联是多对多的，这意味着 Comment 除了外键 `commentable_id`，还保存一个 `commentable` 列。

可以使用_关联作用域_实现多对多关联：

```js
this.Post.hasMany(this.Comment, {
  foreignKey: 'commentable_id',
  scope: {
    commentable: 'post'
  }
});
```

当调用 `post.getComments()`，会自动添加 `WHERE commentable = 'post'`。类似的，当增加新的 comment 到 post， `commentable` 会自动设置为 `'post'`。关联作用域是为了常驻后台，而程序员不需要担心它 - 它无法被禁用. 需要一个完整的多对多的例子，查看[Association scopes](/manual/tutorial/associations.html#scopes)

然后考虑，Post 有一个默认作用域仅展示活动的 post：`where: { active: true }`。这个作用域存在于关联模型（Post）上, 而不是像 `commentable` 这样的关联作用域所做的。就像是调用 `Post.findAll()`会应用默认作用域，调用 `User.getPosts()` 时它同样会被调用  - 这将只向用户返回活动的评论。

为了禁用默认作用域，把 `scope: null` 传递给 getter： `User.getPosts({ scope: null })`。类似的，如果想要应用其他作用域，向 `.scope` 方法传递一个你需要的数组 :
```js
User.getPosts({ scope: ['scope1', 'scope2']});
```

如果你想要在关联模型上创建一个特定作用域的快捷方法，你可以向关联模型传递应用了作用域的模型传递。下面是一个获取所有与一个 user 关联的已被删除的 post 的快捷方式：

```js
const Post = sequelize.define('post', attributes, {
  defaultScope: {
    where: {
      active: true
    }
  },
  scopes: {
    deleted: {
      where: {
        deleted: true
      }
    }
  }
});

User.hasMany(Post); // 常规 getPosts 关联
User.hasMany(Post.scope('deleted'), { as: 'deletedPosts' });

```

```js
User.getPosts(); // WHERE active = true
User.getDeletedPosts(); // WHERE deleted = true
```
