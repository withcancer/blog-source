---
title: sequelizejs中model定义方法
date: 2017-1-21
categories:
- 后端
- sequelizejs
tags:
- 后端
- javascript
- sequelizejs
---

以下是对sequelize.define方法的详解：

# 可以为define方法添加getter,setter
<!-- more -->
``` javascript
const Employee = sequelize.define('employee', {
  name: {
    type: Sequelize.STRING,
    allowNull: false,
    // 字段内的get方法
    get() {
      const title = this.getDataValue('title');
      return this.getDataValue('name') + ' (' + title + ')';
      // this.getDataValue是内建的help方法，用来取得字段的值
    },
  },
  title: {
    type: Sequelize.STRING,
    allowNull: false,
    // 字段内的set方法
    set(val) {
      this.setDataValue('title', val.toUpperCase());
      // 同上
    }
  }
});
```
# 如果要后台检验值时，可以使用model内的validations
``` javascript
// msg属性可以作为错误信息传递到前台
const ValidateMe = sequelize.define('foo', {
  foo: {
    type: Sequelize.STRING,
    validate: {
      is: ["^[a-z]+$",'i'],     // will only allow letters
      is: /^[a-z]+$/i,          // same as the previous example using real RegExp
      not: ["[a-z]",'i'],       // will not allow letters
      isEmail: true,            // checks for email format (foo@bar.com)
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
# 可以在表结构上附加Options
``` javascript
{
  // 不需要updatedAt, createdAt时填false
  timestamps: false,
  // 在删除操作时，并不真正删除，而是设置deletedAt字段。必须和timestamps连用
  paranoid: true,
  // 新加入的字段，例如updatedAt，由驼峰式改为下划线式，updated_at
  underscored: true,
  // 禁止自动更改表名，用默认model名作为表名（即自动由单数转为复数）
  freezeTableName: true,
  // 设置表名
  tableName: 'my_very_custom_table_name',
  // 增加version字段，设为true或者想要的字段名
  version: true
  // 不要createdAt
  createdAt: false,
  // 把updatedAt改成updateTimestamp
  updatedAt: 'updateTimestamp',
  // 同理
  deletedAt: 'destroyTime',
  // engine，设置表引擎
  engine: 'MYISAM'
  // 设置表注释
  comment: "I'm a table comment!",
  // 索引选项，用来生成表时建立索引
   indexes: [
    // Create a unique index on email
    {
      unique: true,
      fields: ['email']
    },

    // Creates a gin index on data with the jsonb_path_ops operator
    {
      fields: ['data'],
      using: 'gin', //PostgreSQL
      operator: 'jsonb_path_ops'
    },

    // By default index name will be [table]_[fields]
    // Creates a multi column partial index
    {
      name: 'public_by_author',
      fields: ['author', 'status'], //聚合索引，带有条件
      where: {
        status: 'public'
      }
    },

    // A BTREE index with a ordered field
    {
      name: 'title_index',
      method: 'BTREE', // 建立一个btree索引
      fields: ['author', {attribute: 'title', collate: 'en_US', order: 'DESC', length: 5}]
    }
  ]

})
```