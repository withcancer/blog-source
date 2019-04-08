---
title: sequelizejs中关系数据的表示
date: 2017-1-22
categories:
- 后端
- sequelizejs
tags:
- 后端
- javascript
- sequelizejs
---
# 外键的四种写法
- user将会添加一个companyId属性，参照Company的主键id
``` javascript
const User = this.sequelize.define('user', {/* attributes */})
const Company  = this.sequelize.define('company', {/* attributes */});

User.belongsTo(Company);
```
<!-- more -->
- 添加company_uuid到user作为user的外键
``` javascript
const User = this.sequelize.define('user', {/* attributes */}, {underscored: true})
const Company  = this.sequelize.define('company', {
  uuid: {
    type: Sequelize.UUID,
    primaryKey: true
  }
});

User.belongsTo(Company);
```
- 添加roleId to user而不是userRoleId
``` javascript
const User = this.sequelize.define('user', {/* attributes */})
const UserRole  = this.sequelize.define('userRole', {/* attributes */});

User.belongsTo(UserRole, {as: 'role'});
```
- 直接指定外键名
``` javascript
const User = this.sequelize.define('user', {/* attributes */})
const Company  = this.sequelize.define('company', {/* attributes */});
// 对source添加
User.belongsTo(Company, {foreignKey: 'fk_company'}); // Adds fk_company to User
```
# one-to-one关系
``` javascript
const User = sequelize.define('user', {/* ... */})
const Project = sequelize.define('project', {/* ... */})

// One-way associations
Project.hasOne(User) // 会对User添加projectId属性，作为User的外键，它将关联Project的主键
```
# many-to-one关系
``` javascript
const User = sequelize.define('user', {/* ... */})
const Project = sequelize.define('project', {/* ... */})

// First let's define a hasMany association
Project.hasMany(User, {as: 'Workers'}) // 添加projectId到User,并且为Project的实例会添加getWorkers和SetWorkers两个方法
```
# one-to-many关系 ON UPDATE CASCADE ON DELETE CASCADE
``` javascript
Project.belongsToMany(User, {through: 'UserProject'}); // through内写model define的名字，而不是变量名，例如Userdefine时写成user，就写user
User.belongsToMany(Project, {through: 'UserProject'}); // userproject,它将关联外键projectid和userId。并且project会添加hasUser,setUser,setUsers,getUser,getUsers方法，User也会得到对应的set,get方法
```
也可以直接在model定义时定义外键