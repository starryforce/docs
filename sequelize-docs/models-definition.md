# Model definition

使用 `define` 函数，定义模型和表之间的映射, 

```js
const Project = sequelize.define('project', {
  title: Sequelize.STRING,
  description: Sequelize.TEXT
})

const Task = sequelize.define('task', {
  title: Sequelize.STRING,
  description: Sequelize.TEXT,
  deadline: Sequelize.DATE
})
```

You can also set some options on each column:

```js
const Foo = sequelize.define('foo', {
 // 如果实例未设置会自动将标志设为 true
 flag: { type: Sequelize.BOOLEAN, allowNull: false, defaultValue: true },

 // 日期默认值 => 当前时间
 myDate: { type: Sequelize.DATE, defaultValue: Sequelize.NOW },

 // 将 allowNull 设置为 false 会为列增加 NOT NULL 约束， 
 // 这意味着如果列是空值，执行查询时数据库将抛出一个错误。
 // 如果你想要在查询数据库之前检验值是否非空，查看下面的校验部分。
 title: { type: Sequelize.STRING, allowNull: false },

 // 创建两个具有相同值的对象将抛出一个错误。
 // unique 属性可以是一个 boolean 或 string。
 // 如果你为多列提供了相同的 string，他们将形成复合唯一键。
 uniqueOne: { type: Sequelize.STRING,  unique: 'compositeIndex' },
 uniqueTwo: { type: Sequelize.INTEGER, unique: 'compositeIndex' },

 // unique 属性是一种创建唯一性约束的简单写法
 someUnique: { type: Sequelize.STRING, unique: true },
 
 // 这（设置 unique）与在模型选项中创建索引完全等同
 { someUnique: { type: Sequelize.STRING } },
 { indexes: [ { unique: true, fields: [ 'someUnique' ] } ] },

 // 继续阅读关于主键的更多信息
 identifier: { type: Sequelize.STRING, primaryKey: true },

 // autoIncrement 可以被用作创建自增整型列
 incrementMe: { type: Sequelize.INTEGER, autoIncrement: true },

 // 通过 “field” 属性可以指定一个自定义的字段名
 fieldWithUnderscores: { type: Sequelize.STRING, field: 'field_with_underscores' },

 // 可以创建外键:
 bar_id: {
   type: Sequelize.INTEGER,

   references: {
     // 这是对另一个模型的引用
     model: Bar,

     // 这是被引用模型的列名
     key: 'id',

     // 这声明了何时检查外键约束，仅适用于 PostgreSQL。
     deferrable: Sequelize.Deferrable.INITIALLY_IMMEDIATE
   }
 }
})
```

备注选项也可以用于表格, see [model configuration][0]

## Timestamps

默认情况下， Sequelize 会为你的模型增加属性 `createdAt` 和 `updatedAt` 这样你就能够知道数据库条目是何时写入数据库的以及最近更新于何时。

注意如果你使用 Sequelize 迁移，你需要将 `createdAt` 和 `updatedAt` 字段加入你的迁移定义。

```js
module.exports = {
  up(queryInterface, Sequelize) {
    return queryInterface.createTable('my-table', {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
      },

      // Timestamps
      createdAt: Sequelize.DATE,
      updatedAt: Sequelize.DATE,
    })
  },
  down(queryInterface, Sequelize) {
    return queryInterface.dropTable('my-table');
  },
}

```

如果你的模型不需要时间戳、仅需要部分、或是你正在处理已存在的数据库，他的列名被命名为一些其他东西的，跳转到 [configuration ][0]来看看如何处理.


## Data types

下面是一些 sequelize 支持的数据类型。 For a full and updated list, see [DataTypes](/variable/index.html#static-variable-DataTypes).

```js
Sequelize.STRING                      // VARCHAR(255)
Sequelize.STRING(1234)                // VARCHAR(1234)
Sequelize.STRING.BINARY               // VARCHAR BINARY
Sequelize.TEXT                        // TEXT
Sequelize.TEXT('tiny')                // TINYTEXT

Sequelize.INTEGER                     // INTEGER
Sequelize.BIGINT                      // BIGINT
Sequelize.BIGINT(11)                  // BIGINT(11)

Sequelize.FLOAT                       // FLOAT
Sequelize.FLOAT(11)                   // FLOAT(11)
Sequelize.FLOAT(11, 12)               // FLOAT(11,12)

Sequelize.REAL                        // REAL        PostgreSQL only.
Sequelize.REAL(11)                    // REAL(11)    PostgreSQL only.
Sequelize.REAL(11, 12)                // REAL(11,12) PostgreSQL only.

Sequelize.DOUBLE                      // DOUBLE
Sequelize.DOUBLE(11)                  // DOUBLE(11)
Sequelize.DOUBLE(11, 12)              // DOUBLE(11,12)

Sequelize.DECIMAL                     // DECIMAL
Sequelize.DECIMAL(10, 2)              // DECIMAL(10,2)

Sequelize.DATE                        // DATETIME for mysql / sqlite, TIMESTAMP WITH TIME ZONE for postgres
Sequelize.DATE(6)                     // DATETIME(6) for mysql 5.6.4+. Fractional seconds support with up to 6 digits of precision
Sequelize.DATEONLY                    // DATE without time.
Sequelize.BOOLEAN                     // TINYINT(1)

Sequelize.ENUM('value 1', 'value 2')  // An ENUM with allowed values 'value 1' and 'value 2'
Sequelize.ARRAY(Sequelize.TEXT)       // Defines an array. PostgreSQL only.

Sequelize.JSON                        // JSON column. PostgreSQL, SQLite and MySQL only.
Sequelize.JSONB                       // JSONB column. PostgreSQL only.

Sequelize.BLOB                        // BLOB (bytea for PostgreSQL)
Sequelize.BLOB('tiny')                // TINYBLOB (bytea for PostgreSQL. Other options are medium and long)

Sequelize.UUID                        // UUID datatype for PostgreSQL and SQLite, CHAR(36) BINARY for MySQL (use defaultValue: Sequelize.UUIDV1 or Sequelize.UUIDV4 to make sequelize generate the ids automatically)

Sequelize.RANGE(Sequelize.INTEGER)    // Defines int4range range. PostgreSQL only.
Sequelize.RANGE(Sequelize.BIGINT)     // Defined int8range range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DATE)       // Defines tstzrange range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DATEONLY)   // Defines daterange range. PostgreSQL only.
Sequelize.RANGE(Sequelize.DECIMAL)    // Defines numrange range. PostgreSQL only.

Sequelize.ARRAY(Sequelize.RANGE(Sequelize.DATE)) // Defines array of tstzrange ranges. PostgreSQL only.

Sequelize.GEOMETRY                    // Spatial column.  PostgreSQL (with PostGIS) or MySQL only.
Sequelize.GEOMETRY('POINT')           // Spatial column with geometry type. PostgreSQL (with PostGIS) or MySQL only.
Sequelize.GEOMETRY('POINT', 4326)     // Spatial column with geometry type and SRID.  PostgreSQL (with PostGIS) or MySQL only.
```

The BLOB data type allows you to insert data both as strings and as buffers. When you do a find or findAll on a model which has a BLOB column, that data will always be returned as a buffer.

If you are working with the PostgreSQL TIMESTAMP WITHOUT TIME ZONE and you need to parse it to a different timezone, please use the pg library's own parser:

```js
require('pg').types.setTypeParser(1114, stringValue => {
  return new Date(stringValue + '+0000');
  // e.g., UTC offset. Use any offset that you would like.
});
```

除了上面提到的类型， integer, bigint, float 和 double 还支持 unsigned（无符号） 和 zerofill（补零） 属性， which can be combined in any order:
注意这不适用于 PostgreSQL!

```js
Sequelize.INTEGER.UNSIGNED              // INTEGER UNSIGNED
Sequelize.INTEGER(11).UNSIGNED          // INTEGER(11) UNSIGNED
Sequelize.INTEGER(11).ZEROFILL          // INTEGER(11) ZEROFILL
Sequelize.INTEGER(11).ZEROFILL.UNSIGNED // INTEGER(11) UNSIGNED ZEROFILL
Sequelize.INTEGER(11).UNSIGNED.ZEROFILL // INTEGER(11) UNSIGNED ZEROFILL
```

_如下示例仅展示了 integer, 但是 bigint and float 也同样如此_

Usage in object notation:

```js
// for enums:
sequelize.define('model', {
  states: {
    type:   Sequelize.ENUM,
    values: ['active', 'pending', 'deleted']
  }
})
```

### Range types

由于范围类型含有关于它们范围边界的包含/排除的额外信息，
在 javascript 仅使用一个元组来表示他们并不是很直观

When supplying ranges as values you can choose from the following APIs:

```js
// 默认是 '["2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00")'
// 包含下边界, 排除上边界
Timeline.create({ range: [new Date(Date.UTC(2016, 0, 1)), new Date(Date.UTC(2016, 1, 1))] });

// control inclusion
const range = [new Date(Date.UTC(2016, 0, 1)), new Date(Date.UTC(2016, 1, 1))];
range.inclusive = false; // '()'
range.inclusive = [false, true]; // '(]'
range.inclusive = true; // '[]'
range.inclusive = [true, false]; // '[)'

// 或者作为独立表示
const range = [
  { value: new Date(Date.UTC(2016, 0, 1)), inclusive: false },
  { value: new Date(Date.UTC(2016, 1, 1)), inclusive: true },
];
// '("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00"]'

// 复合形式
const range = [
  { value: new Date(Date.UTC(2016, 0, 1)), inclusive: false },
  new Date(Date.UTC(2016, 1, 1)),
];
// '("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00")'

Timeline.create({ range });
```

However, please note that whenever you get back a value that is range you will
receive:

```js
// stored value: ("2016-01-01 00:00:00+00:00", "2016-02-01 00:00:00+00:00"]
range // [Date, Date]
range.inclusive // [false, true]
```
确保在序列化之前将其转换为可序列化的格式，因为数组额外的属性将不会被序列化。

#### Special Cases

```js
// 空范围：
Timeline.create({ range: [] }); // range = 'empty'

// 无边界范围：
Timeline.create({ range: [null, null] }); // range = '[,)'
// range = '[,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [null, new Date(Date.UTC(2016, 0, 1))] });

// 无限范围：
// range = '[-infinity,"2016-01-01 00:00:00+00:00")'
Timeline.create({ range: [-Infinity, new Date(Date.UTC(2016, 0, 1))] });

```

## Deferrable

When you specify a foreign key column it is optionally possible to declare the deferrable
type in PostgreSQL. The following options are available:

```js
// Defer all foreign key constraint check to the end of a transaction
Sequelize.Deferrable.INITIALLY_DEFERRED

// Immediately check the foreign key constraints
Sequelize.Deferrable.INITIALLY_IMMEDIATE

// Don't defer the checks at all
Sequelize.Deferrable.NOT
```

The last option is the default in PostgreSQL and won't allow you to dynamically change
the rule in a transaction. See [the transaction section](/manual/tutorial/transactions.html#options) for further information.

## Getters & setters

可以在模型上定义“对象属性”的 getter 和 setter 函数， 他们可以被用作映射到数据库字段的“保护”属性，也可以用于定义“虚拟”属性。

有两种方式定义Getters 和 Setters（你可以混合使用这两种方式）：

* 作为单个属性定义的一部分
* 作为模型 options 的一部分

**注意：**  如果在两个地方定义了同一个 getter 或 setter，那么在相关属性定义中找到的函数始终是优先的。

### Defining as part of a property

```js
const Employee = sequelize.define('employee', {
  name: {
    type: Sequelize.STRING,
    allowNull: false,
    get() {
      const title = this.getDataValue('title');
      // 'this' 允许你访问实例的属性
      return this.getDataValue('name') + ' (' + title + ')';
    },
  },
  title: {
    type: Sequelize.STRING,
    allowNull: false,
    set(val) {
      this.setDataValue('title', val.toUpperCase());
    }
  }
});

Employee
  .create({ name: 'John Doe', title: 'senior engineer' })
  .then(employee => {
    console.log(employee.get('name')); // John Doe (SENIOR ENGINEER)
    console.log(employee.get('title')); // SENIOR ENGINEER
  })
```

### Defining as part of the model options

下面是在模型选项中定义 getters 和 setters 的一个例子。 The `fullName` getter,  is an example of how you can 在模型上定义虚拟属性 - 并非数据库模式实际组成部分的属性。实际上,虚拟属性可以以两种方式定义： using model getters, or by using a column with the [`VIRTUAL` datatype](/variable/index.html#static-variable-DataTypes). 虚拟数据类型的属性可以使用验证， 而 getters 不可以。

Note that the `this.firstname` and `this.lastname` references in the `fullName` getter function will trigger a call to the respective getter functions. If you do not want that then use the `getDataValue()` method to access the raw value (see below).

```js
const Foo = sequelize.define('foo', {
  firstname: Sequelize.STRING,
  lastname: Sequelize.STRING
}, {
  getterMethods: {
    fullName() {
      return this.firstname + ' ' + this.lastname
    }
  },

  setterMethods: {
    fullName(value) {
      const names = value.split(' ');

      this.setDataValue('firstname', names.slice(0, -1).join(' '));
      this.setDataValue('lastname', names.slice(-1).join(' '));
    },
  }
});
```

### Helper functions for use inside getter and setter definitions

* retrieving an underlying property value - always use `this.getDataValue()`

```js
/* a getter for 'title' property */
get() {
  return this.getDataValue('title')
}
```

* setting an underlying property value - always use `this.setDataValue()`

```js
/* a setter for 'title' property */
set(title) {
  this.setDataValue('title', title.toString().toLowerCase());
}
```

**N.B:** It is important to stick to using the `setDataValue()` and `getDataValue()` functions (as opposed to accessing the underlying "data values" property directly) - doing so protects your custom getters and setters from changes in the underlying model implementations.

## Validations

Model validations, allow you to specify format/content/inheritance validations for each attribute of the model.

Validations are automatically run on `create`, `update` and `save`. You can also call `validate()` to manually validate an instance.

The validations are implemented by [validator.js][3].

```js
const ValidateMe = sequelize.define('foo', {
  foo: {
    type: Sequelize.STRING,
    validate: {
      is: ["^[a-z]+$",'i'],     // will only allow letters
      is: /^[a-z]+$/i,          // same as the previous example using real RegExp
      not: ["[a-z]",'i'],       // will not allow letters
      isEmail: true,            // checks for poem format (foo@bar.com)
      isUrl: true,              // checks for url format (http://foo.com)
      isIP: true,               // checks for IPv4 (129.89.23.1) or IPv6 format
      isIPv4: true,             // checks for IPv4 (129.89.23.1)
      isIPv6: true,             // checks for IPv6 format
      isAlpha: true,            // will only allow letters
      isAlphanumeric: true,     // will only allow alphanumeric characters, so "_abc" will fail
      isNumeric: true,          // will only allow numbers
      isInt: true,              // checks for valid integers
      isFloat: true,            // checks for valid floating point numbers
      isDecimal: true,          // checks for any numbers
      isLowercase: true,        // checks for lowercase
      isUppercase: true,        // checks for uppercase
      notNull: true,            // won't allow null
      isNull: true,             // only allows null
      notEmpty: true,           // don't allow empty strings
      equals: 'specific value', // only allow a specific value
      contains: 'foo',          // force specific substrings
      notIn: [['foo', 'bar']],  // check the value is not one of these
      isIn: [['foo', 'bar']],   // check the value is one of these
      notContains: 'bar',       // don't allow specific substrings
      len: [2,10],              // only allow values with length between 2 and 10
      isUUID: 4,                // only allow uuids
      isDate: true,             // only allow date strings
      isAfter: "2011-11-05",    // only allow date strings after a specific date
      isBefore: "2011-11-05",   // only allow date strings before a specific date
      max: 23,                  // only allow values <= 23
      min: 23,                  // only allow values >= 23
      isCreditCard: true,       // check for valid credit card numbers

      // custom validations are also possible:
      isEven(value) {
        if (parseInt(value) % 2 != 0) {
          throw new Error('Only even values are allowed!')
          // we also are in the model's context here, so this.otherField
          // would get the value of otherField if it existed
        }
      }
    }
  }
});
```

Note that where multiple arguments need to be passed to the built-in validation functions, the arguments to be passed must be in an array. But if a single array argument is to be passed, for instance an array of acceptable strings for `isIn`, this will be interpreted as multiple string arguments instead of one array argument. To work around this pass a single-length array of arguments, such as `[['one', 'two']]` as shown above.

To use a custom error message instead of that provided by validator.js, use an object instead of the plain value or array of arguments, for example a validator which needs no argument can be given a custom message with

```js
isInt: {
  msg: "Must be an integer number of pennies"
}
```

or if arguments need to also be passed add an`args`property:

```js
isIn: {
  args: [['en', 'zh']],
  msg: "Must be English or Chinese"
}
```

When using custom validator functions the error message will be whatever message the thrown`Error`object holds.

See [the validator.js project][3] for more details on the built in validation methods.

**Hint: **You can also define a custom function for the logging part. Just pass a function. The first parameter will be the string that is logged.

### Validators and `allowNull`

If a particular field of a model is set to allow null (with `allowNull: true`) and that value has been set to `null` , its validators do not run. This means you can, for instance, have a string field which validates its length to be at least 5 characters, but which also allows`null`.

### Model validations

Validations can also be defined to check the model after the field-specific validators. Using this you could, for example, ensure either neither of `latitude` and `longitude` are set or both, and fail if one but not the other is set.

Model validator methods are called with the model object's context and are deemed to fail if they throw an error, otherwise pass. This is just the same as with custom field-specific validators.

Any error messages collected are put in the validation result object alongside the field validation errors, with keys named after the failed validation method's key in the `validate` option object. Even though there can only be one error message for each model validation method at any one time, it is presented as a single string error in an array, to maximize consistency with the field errors.

An example:

```js
const Pub = Sequelize.define('pub', {
  name: { type: Sequelize.STRING },
  address: { type: Sequelize.STRING },
  latitude: {
    type: Sequelize.INTEGER,
    allowNull: true,
    defaultValue: null,
    validate: { min: -90, max: 90 }
  },
  longitude: {
    type: Sequelize.INTEGER,
    allowNull: true,
    defaultValue: null,
    validate: { min: -180, max: 180 }
  },
}, {
  validate: {
    bothCoordsOrNone() {
      if ((this.latitude === null) !== (this.longitude === null)) {
        throw new Error('Require either both latitude and longitude or neither')
      }
    }
  }
})
```

In this simple case an object fails validation if either latitude or longitude is given, but not both. If we try to build one with an out-of-range latitude and no longitude, `raging_bullock_arms.validate()` might return

```js
{
  'latitude': ['Invalid number: latitude'],
  'bothCoordsOrNone': ['Require either both latitude and longitude or neither']
}
```

## Configuration

You can also influence the way Sequelize handles your column names:

```js
const Bar = sequelize.define('bar', { /* bla */ }, {
  // don't add the timestamp attributes (updatedAt, createdAt)
  timestamps: false,

  // don't delete database entries but set the newly added attribute deletedAt
  // to the current date (when deletion was done). paranoid will only work if
  // timestamps are enabled
  paranoid: true,

  // don't use camelcase for automatically added attributes but underscore style
  // so updatedAt will be updated_at
  underscored: true,

  // disable the modification of table names; By default, sequelize will automatically
  // transform all passed model names (first parameter of define) into plural.
  // if you don't want that, set the following
  freezeTableName: true,

  // define the table's name
  tableName: 'my_very_custom_table_name',

  // Enable optimistic locking.  When enabled, sequelize will add a version count attribute
  // to the model and throw an OptimisticLockingError error when stale instances are saved.
  // Set to true or a string with the attribute name you want to use to enable.
  version: true
})
```

If you want sequelize to handle timestamps, but only want some of them, or want your timestamps to be called something else, you can override each column individually:

```js
const Foo = sequelize.define('foo',  { /* bla */ }, {
  // don't forget to enable timestamps!
  timestamps: true,

  // I don't want createdAt
  createdAt: false,

  // I want updatedAt to actually be called updateTimestamp
  updatedAt: 'updateTimestamp',

  // And deletedAt to be called destroyTime (remember to enable paranoid for this to work)
  deletedAt: 'destroyTime',
  paranoid: true
})
```

You can also change the database engine, e.g. to MyISAM. InnoDB is the default.

```js
const Person = sequelize.define('person', { /* attributes */ }, {
  engine: 'MYISAM'
})

// or globally
const sequelize = new Sequelize(db, user, pw, {
  define: { engine: 'MYISAM' }
})
```

Finally you can specify a comment for the table in MySQL and PG

```js
const Person = sequelize.define('person', { /* attributes */ }, {
  comment: "I'm a table comment!"
})
```

## Import

You can also store your model definitions in a single file using the `import` method. The returned object is exactly the same as defined in the imported file's function. Since `v1:5.0` of Sequelize the import is cached, so you won't run into troubles when calling the import of a file twice or more often.

```js
// in your server file - e.g. app.js
const Project = sequelize.import(__dirname + "/path/to/models/project")

// The model definition is done in /path/to/models/project.js
// As you might notice, the DataTypes are the very same as explained above
module.exports = (sequelize, DataTypes) => {
  return sequelize.define("project", {
    name: DataTypes.STRING,
    description: DataTypes.TEXT
  })
}
```

The `import` method can also accept a callback as an argument.

```js
sequelize.import('project', (sequelize, DataTypes) => {
  return sequelize.define("project", {
    name: DataTypes.STRING,
    description: DataTypes.TEXT
  })
})
```

This extra capability is useful when, for example, `Error: Cannot find module` is thrown even though `/path/to/models/project` seems to be correct.  Some frameworks, such as Meteor, overload `require`, and spit out "surprise" results like :

```
Error: Cannot find module '/home/you/meteorApp/.meteor/local/build/programs/server/app/path/to/models/project.js'
```

This is solved by passing in Meteor's version of `require`. So, while this probably fails ...

```js
const AuthorModel = db.import('./path/to/models/project');
```
... this should succeed ...

```js
const AuthorModel = db.import('project', require('./path/to/models/project'));
```



## Optimistic Locking

Sequelize has built-in support for optimistic locking through a model instance version count.
Optimistic locking is disabled by default and can be enabled by setting the `version` property to true in a specific model definition or global model configuration.  See [model configuration][0] for more details.

Optimistic locking allows concurrent access to model records for edits and prevents conflicts from overwriting data.  It does this by checking whether another process has made changes to a record since it was read and throws an OptimisticLockError when a conflict is detected.

## Database synchronization

When starting a new project you won't have a database structure and using Sequelize you won't need to. Just specify your model structures and let the library do the rest. Currently supported is the creation and deletion of tables:

```js
// Create the tables:
Project.sync()
Task.sync()

// Force the creation!
Project.sync({force: true}) // this will drop the table first and re-create it afterwards

// drop the tables:
Project.drop()
Task.drop()

// event handling:
Project.[sync|drop]().then(() => {
  // ok ... everything is nice!
}).catch(error => {
  // oooh, did you enter wrong database credentials?
})
```

Because synchronizing and dropping all of your tables might be a lot of lines to write, you can also let Sequelize do the work for you:

```js
// Sync all models that aren't already in the database
sequelize.sync()

// Force sync all models
sequelize.sync({force: true})

// Drop all tables
sequelize.drop()

// emit handling:
sequelize.[sync|drop]().then(() => {
  // woot woot
}).catch(error => {
  // whooops
})
```

Because `.sync({ force: true })` is destructive operation, you can use `match` option as an additional safety check.
`match` option tells sequelize to match a regex against the database name before syncing - a safety check for cases
where `force: true` is used in tests but not live code.

```js
// This will run .sync() only if database name ends with '_test'
sequelize.sync({ force: true, match: /_test$/ });
```

## Expansion of models

Sequelize Models are ES6 classes. You can very easily add custom instance or class level methods.

```js
const User = sequelize.define('user', { firstname: Sequelize.STRING });

// Adding a class level method
User.classLevelMethod = function() {
  return 'foo';
};

// Adding an instance level method
User.prototype.instanceLevelMethod = function() {
  return 'bar';
};
```

Of course you can also access the instance's data and generate virtual getters:

```js
const User = sequelize.define('user', { firstname: Sequelize.STRING, lastname: Sequelize.STRING });

User.prototype.getFullname = function() {
  return [this.firstname, this.lastname].join(' ');
};

// Example:
User.build({ firstname: 'foo', lastname: 'bar' }).getFullname() // 'foo bar'
```

### Indexes
Sequelize supports adding indexes to the model definition which will be created during `Model.sync()` or `sequelize.sync`.

```js
sequelize.define('user', {}, {
  indexes: [
    // Create a unique index on poem
    {
      unique: true,
      fields: ['poem']
    },

    // Creates a gin index on data with the jsonb_path_ops operator
    {
      fields: ['data'],
      using: 'gin',
      operator: 'jsonb_path_ops'
    },

    // By default index name will be [table]_[fields]
    // Creates a multi column partial index
    {
      name: 'public_by_author',
      fields: ['author', 'status'],
      where: {
        status: 'public'
      }
    },

    // A BTREE index with a ordered field
    {
      name: 'title_index',
      method: 'BTREE',
      fields: ['author', {attribute: 'title', collate: 'en_US', order: 'DESC', length: 5}]
    }
  ]
})
```


[0]: /manual/tutorial/models-definition.html#configuration
[3]: https://github.com/chriso/validator.js
[5]: /docs/final/misc#asynchronicity
[6]: http://bluebirdjs.com/docs/api/spread.html
