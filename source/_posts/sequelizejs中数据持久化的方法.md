---
title: sequelizejs中数据持久化的方法
date: 2017-1-21
categories:
- 后端
- sequelizejs
tags:
- 后端
- javascript
- sequelizejs
---
# sync
只能用来建表
# save
``` javascript
Task
  .build({ title: 'foo', description: 'bar', deadline: new Date() })
  .save()
  .then(anotherTask => {
    // you can now access the currently saved task with the variable anotherTask... nice!
  })
  .catch(error => {
    // Ooops, do some error-handling
  })
```
<!-- more -->
# create 相当于结合了build和save
``` javascript
Task.create({ title: 'foo', description: 'bar', deadline: new Date() }).then(task => {
  // you can now access the newly created task via the variable task
})
```
# 多条create
``` javascript
User.bulkCreate([
  { username: 'barfooz', isAdmin: true },
  { username: 'foo', isAdmin: true },
  { username: 'bar', isAdmin: false }
]).then(() => { // Notice: There are no arguments here, as of right now you'll have to...
  return User.findAll();
}).then(users => {
  console.log(users) // ... in order to get the array of user objects
})
```
# update的三种形式
``` javascript
task.save({fields: ['title']}).then(() => {
 // title will now be 'foooo' but description is the very same as before
})

// The equivalent call using update looks like this:
task.update({ title: 'foooo', description: 'baaaaaar'}, {fields: ['title']}).then(() => {
 // title will now be 'foooo' but description is the very same as before
})
// 先查出来，然后reload
Person.findOne({ where: { name: 'john' } }).then(person => {
  person.name = 'jane'
  console.log(person.name) // 'jane'

  person.reload().then(() => {
    console.log(person.name) // 'john'
  })
})
```

# delete
``` javascript
task.destroy({force: true})
```