---
title: sequelizejs中数据查询的方法
date: 2017-1-22
categories:
- 后端
- sequelizejs
tags:
- 后端
- javascript
- sequelizejs
---
# Raw query
- 最简单的查询
``` javascript
sequelize.query("SELECT * FROM myTable").then(myTableRows => {
  console.log(myTableRows)
})
```

<!-- more -->
- Map to model
``` javascript
sequelize
  .query('SELECT * FROM projects', { model: Projects })
  .then(projects => {
    // Each record will now be mapped to the project's model.
    console.log(projects)
  })
```

- Options
``` javascript
sequelize
  .query('SELECT 1', {
    // 可以设为false或者console
    logging: console.log,

    // 是否只返回结果集中的第一行
    plain: false,
    // 如果没有model可以映射，设为false
    raw: false,

    // 设置了queryType后，可以改变返回结果的形式
    type: Sequelize.QueryTypes.SELECT
  })
```

- 参数查询
``` javascript
  sequelize
  .query(
    'SELECT * FROM projects WHERE status = ?',
    { raw: true, replacements: ['active']
  )
  .then(projects => {
    console.log(projects)
  })
```

# Model Query
- findById
``` javascript
Project.findById(123).then(project => {
  // project will be an instance of Project and stores the content of the table entry
  // with id 123. if such an entry is not defined you will get null
})
```
- findByAttribute
``` javascript
// search for attributes
Project.findOne({ where: {title: 'aProject'} }).then(project => {
  // project will be the first entry of the Projects table with the title 'aProject' || null
})
```
- findOrCreate 相当于mongodb的upsert
``` javascript
User
  .findOrCreate({where: {username: 'sdepold'}, defaults: {job: 'Technical Lead JavaScript'}})
  .spread((user, created) => { // 返回object和boolean
    console.log(user.get({
      plain: true
    }))
    console.log(created)
```
- findAndCountAll
对结果集进行偏移或约束
``` javascript
Project
  .findAndCountAll({
     where: {
        title: {
          [Op.like]: 'foo%'
        }
     },
     offset: 10,
     limit: 2
  })
```
- findAll
约束：
``` javascript
User.findAll({ offset: 10, limit: 2 , order: 'title DESC', group: 'name');
```
- 操作符,这些操作符也可以进一步组合
``` javascript
[Op.and]: {a: 5},           // AND (a = 5)
[Op.or]: [{a: 5}, {a: 6}],  // (a = 5 OR a = 6)
[Op.gt]: 6,                // id > 6
[Op.gte]: 6,               // id >= 6
[Op.lt]: 10,               // id < 10
[Op.lte]: 10,              // id <= 10
[Op.ne]: 20,               // id != 20
[Op.between]: [6, 10],     // BETWEEN 6 AND 10
[Op.notBetween]: [11, 15], // NOT BETWEEN 11 AND 15
[Op.in]: [1, 2],           // IN [1, 2]
[Op.notIn]: [1, 2],        // NOT IN [1, 2]
[Op.like]: '%hat',         // LIKE '%hat'
[Op.notLike]: '%hat',       // NOT LIKE '%hat'
[Op.iLike]: '%hat',         // ILIKE '%hat' (case insensitive)  (PG only)
[Op.notILike]: '%hat',      // NOT ILIKE '%hat'  (PG only)
[Op.overlap]: [1, 2],       // && [1, 2] (PG array overlap operator)
[Op.contains]: [1, 2],      // @> [1, 2] (PG array contains operator)
[Op.contained]: [1, 2],     // <@ [1, 2] (PG array contained by operator)
[Op.any]: [2,3]            // ANY ARRAY[2, 3]::INTEGER (PG only)
```
