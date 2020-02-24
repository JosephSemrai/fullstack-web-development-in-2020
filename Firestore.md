# Introduction

https://i.imgur.com/sTfQeI4.png

https://i.imgur.com/UcXmRGt.png



Read the documentation. Summarize it here, then build the profile app here.

# Firestore

When using Firestore instead of Mongoose, we are truly exposed to NoSQL in that we are not attempting to emulate a relational database with an ODM (object-document mapper) such as Mongoose. Here, 

As our database designs grow in complexity, it is important to realize that indexing and searching inside deeply nested documents (especially with arrays) quickly becomes very expensive. This is because we need to download the entire object before accessing a particular nested piece of data.

With this, we can **denormalize** our data (flatten data structures)

