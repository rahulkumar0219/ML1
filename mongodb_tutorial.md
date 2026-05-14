# MongoDB Tutorial — Student Database

## Database Setup

```javascript
use school
```

Fields used: `name`, `age`, `roll`, `scores`, `subjects`

---

## 1. Inserting Documents

### Single Insert

```javascript
db.students.insertOne({
  name: "Alice",
  age: 20,
  roll: 101,
  scores: { math: 95, science: 88, english: 92 },
  subjects: ["Math", "Science", "English"]
})
```

### Batch Insert

```javascript
db.students.insertMany([
  {
    name: "Bob",
    age: 22,
    roll: 102,
    scores: { math: 75, science: 60, english: 80 },
    subjects: ["Math", "Science"]
  },
  {
    name: "Carol",
    age: 21,
    roll: 103,
    scores: { math: 91, science: 85, english: 97 },
    subjects: ["Math", "Science", "English"]
  },
  {
    name: "David",
    age: 19,
    roll: 104,
    scores: { math: 60, english: 70 },
    subjects: ["Math", "English"]
  },
  {
    name: "Eva",
    age: 23,
    roll: 105,
    scores: { science: 78, english: 83 },
    subjects: ["Science", "English"]
  },
  {
    name: "Frank",
    age: 20,
    roll: 106,
    scores: { math: 55 },
    subjects: ["Math"]
  }
])
```

### Insert Validation

```javascript
db.createCollection("students", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "age", "roll"],
      properties: {
        name: { bsonType: "string",  description: "Must be a string" },
        age:  { bsonType: "int",     minimum: 16, description: "Must be >= 16" },
        roll: { bsonType: "int",     description: "Must be an integer" }
      }
    }
  },
  validationAction: "error"
})

// FAILS — age is a string, not int
db.students.insertOne({ name: "Bad", age: "twenty", roll: 107 })

// FAILS — roll is missing
db.students.insertOne({ name: "NoRoll", age: 18 })
```

---

## 2. Removing Documents

### Remove one document

```javascript
db.students.deleteOne({ name: "Frank" })
```

### Remove multiple documents

```javascript
// Remove all students younger than 20
db.students.deleteMany({ age: { $lt: 20 } })
```

---

## 3. Updating Documents

### Document Replacement

Replaces the entire document (keeps the same `_id`):

```javascript
db.students.replaceOne(
  { roll: 102 },
  {
    name: "Bob",
    age: 23,
    roll: 102,
    scores: { math: 88, science: 79, english: 85 },
    subjects: ["Math", "Science", "English"]
  }
)
```

### Using Modifiers

```javascript
// $set — update a specific field
db.students.updateOne(
  { roll: 101 },
  { $set: { age: 21, "scores.math": 99 } }
)

// $inc — increment age by 1
db.students.updateOne(
  { roll: 104 },
  { $inc: { age: 1 } }
)

// $unset — remove the english score
db.students.updateOne(
  { roll: 103 },
  { $unset: { "scores.english": "" } }
)

// $push — add a subject to the array
db.students.updateOne(
  { roll: 105 },
  { $push: { subjects: "Math" } }
)
```

### Upsert (insert if not found)

```javascript
db.students.updateOne(
  { roll: 110 },
  {
    $set: {
      name: "Harry",
      age: 21,
      roll: 110,
      scores: { math: 95, science: 70 },
      subjects: ["Math", "Science"]
    }
  },
  { upsert: true }
)
```

### Update Multiple Documents

```javascript
// Add a "status" field to all students aged 21 or above
db.students.updateMany(
  { age: { $gte: 21 } },
  { $set: { status: "senior" } }
)
```

### Return Updated Document

```javascript
// Returns the document AFTER update
db.students.findOneAndUpdate(
  { roll: 101 },
  { $set: { "scores.math": 100 } },
  { returnDocument: "after" }
)
```

---

## 4. Queries (10 Examples)

### Query 1 — Find all documents

```javascript
db.students.find({})
```

---

### Query 2 — findOne by specific value

```javascript
db.students.findOne({ roll: 101 })
```

---

### Query 3 — Find by name

```javascript
db.students.find({ name: "Carol" })
```

---

### Query 4 — Query Conditionals

```javascript
// Age between 20 and 22
db.students.find({ age: { $gte: 20, $lte: 22 } })

// Math score greater than 80
db.students.find({ "scores.math": { $gt: 80 } })
```

---

### Query 5 — OR Query

```javascript
// Students aged 20 OR who take English
db.students.find({
  $or: [
    { age: 20 },
    { subjects: "English" }
  ]
})
```

---

### Query 6 — $not Operator

```javascript
// Students whose age is NOT less than 21
db.students.find({
  age: { $not: { $lt: 21 } }
})
```

---

### Query 7 — Null Values

```javascript
// Students where science score is missing or null
db.students.find({ "scores.science": null })
```

---

### Query 8 — Regular Expression

```javascript
// Students whose name starts with "A" (case-insensitive)
db.students.find({ name: /^a/i })

// Students whose name contains the letter "o"
db.students.find({ name: /o/ })
```

---

### Query 9 — Querying Arrays

```javascript
// Students who take Math
db.students.find({ subjects: "Math" })

// Students who take BOTH Math AND Science
db.students.find({ subjects: { $all: ["Math", "Science"] } })

// Students taking exactly 1 subject
db.students.find({ subjects: { $size: 1 } })
```

---

### Query 10 — $where Query

```javascript
// Students older than 20 using a JavaScript expression
db.students.find({
  $where: function() {
    return this.age > 20;
  }
})

// Students whose roll number is greater than their age (string form)
db.students.find({ $where: "this.roll > this.age" })
```

> **Note:** `$where` runs JavaScript on every document and is slower than standard operators. Use it only when needed.

---

## Quick Reference

| Operation | Command |
|-----------|---------|
| Insert one | `insertOne({...})` |
| Insert many | `insertMany([...])` |
| Delete one | `deleteOne({filter})` |
| Delete many | `deleteMany({filter})` |
| Replace doc | `replaceOne({filter}, {newDoc})` |
| Update fields | `updateOne({filter}, {$set:{...}})` |
| Upsert | `updateOne({filter}, {...}, {upsert:true})` |
| Update many | `updateMany({filter}, {$set:{...}})` |
| Find all | `find({})` |
| Find one | `findOne({filter})` |
| Conditional | `find({field: {$gt: val}})` |
| OR query | `find({$or:[...]})` |
| NOT | `find({field: {$not:{...}}})` |
| Null check | `find({field: null})` |
| Regex | `find({field: /pattern/})` |
| Array match | `find({arr: "value"})` |
| $where | `find({$where: "..."})` |
