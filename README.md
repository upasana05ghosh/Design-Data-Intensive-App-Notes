# Design-Data-Intensive-App-Notes
Design Data Intensive App Notes

# Chapter 1: Reliable, Scalable & Maintainable

### Reliable
 * App performs as expected
 * Can tolerate user mistake
 * Performance is good
 * System prevents unauthorize access and abuse

### Scalability
  * Ability to cope with increase load
  * Describe load
      * DB -> Ratio of R/w
      * Server -> Request/sec
      * Cache -> Hit rate
      * Online server -> response time
           * P50 -> half of user receives request
       
### Maintainability
  * Operability -> good monitor and documentation
  * Simplicity -> easy to understand
  * Evolvability -> Easy to make new changes

---
# Chapter 2: Data Models and Query Languages

| SQL | NOSQL   | 
| :---:   | :---: |
| Relational | JSON |

### Advantage
| SQL | NOSQL   | 
| :---:   | :---: |
| Query optimiser is available | Schema flexibility |
| Better support for n:1, n:m relation | Better performance due to locality|
|  |Close to data structure for mapping|

### Disadvantage
| SQL | NOSQL   | 
| :---:   | :---: |
| Schema change is long and slow | Week support for join |
| Better support for n:1, n:m relation | Difficult with n:m relation|
|  |On Update, entire document update|

### Conclusion
Go with hybrid

---
