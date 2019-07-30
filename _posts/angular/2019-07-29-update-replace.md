---
title: "Differenct between update and replace"
date: 2019-07-29 12:26:00 +0900
comments: true
categories: mongodb
tags: [update, replace]
---



mongoDB에서 update와 replace의 차이를 비교해보겠습니다.
한국어로는 관련 내용이 없어서 stackoverflow를 참고하여 정리하였습니다.




### update

```
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)


db.collection.updateOne(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)


db.collection.updateMany(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
```

update는 Update, One, Many 세 가지 함수가 있습니다.<br/>
모두 데이터를 갱신하는 역할을 하며 update에만 multi 여부를 묻고 다른 함수는 multi 여부가 없습니다.<br/>
즉, update는 multi의 true/false 여부에 따라 updateOne이나 updateMany 모두 구현 가능합니다.

<p>
update는 우리가 일반적으로 아는 DB의 update와 같습니다. <br/>
존재하는 데이터의 항목의 값을 변경하는 것입니다.<br/>
여기에 만일 존재하지 않으면 추가하는 항목인 upsert 와 filter 등의 옵션도 제공하고 있습니다.


예를 들어 봅시다.


```json
{
id: 12345,
name: "test",
age: 1
}
```


위의 데이터에서 age를 10살로 갱신(update)하려면, {$set}을 사용해야 합니다.


```
db.collection.update( {id: 12345}, {$set: {age: 10}}, {multi: false})
```


id:12345의 age를 10으로 set 하게 됩니다.


```json
{
id: 12345,
name: "test",
age: 10
}
```


filter에는 검색 때와 같이 $gte $lte 도 사용할 수 있으며 (이때는 {}multi: true}로 설정), $unset은 항목을 제거하는 것입니다.


### replace

버전 3.2부터 replaceOne이 적용되었습니다.

```
db.collection.replaceOne(
   <filter>,
   <replacement>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>
   }
)
```

replace는 replaceOne 밖에 없습니다. 즉, 여러 데이터를 한번에 replace 할 수는 없다는 점이 update하고 다른 점입니다.
<br/>
또 하나의 다른 점은 데이터를 완전히 변경한다는 점입니다. 즉, update는 해당 필드만 수정하는데에 반해 replaceOne은 해당 필드로 데이터를 완전히 바꾸어 버립니다. 


예를 들어 설명해보면,

```json
{
id: 12345,
name: "test",
age: 1
}
```
 
위의 데이터에서 age를 10살로 변경(replace) 해보겠습니다. $set을 사용할 필요가 없습니다.

```
db.collection.replaceOne( {id: 12345}, {age: 10}};
```

id:12345의 데이터는 age: 10으로 변경되었습니다.

```json
{
id: 12345,
age: 10
}
```

기존 버전에서는 애매했던 update와 replace간의 역할이 3.2 버전부터는 보다 명확해졌습니다. 따라서,
목적에 맞게 데이터를 갱신하려면 update를, 데이터를 변경하려면 replace를 사용하여야 합니다. 

끝.