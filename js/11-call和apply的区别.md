1. Function.prototype.apply和Function.prototype.call 的作用是一样的，区别在于传入参数的不同；
2. 第一个参数都是，指定函数体内this的指向；
3. 第二个参数开始不同，apply是传入带下标的集合，数组或者类数组，apply把它传给函数作为参数，call从第二个开始传入的参数是不固定的，都会传给函数作为参数。

大家好像不喜欢看ECMA草案。

[call:](https://tc39.es/ecma262/#sec-function.prototype.call)
[![image](https://user-images.githubusercontent.com/25839518/76718895-13d75300-6773-11ea-88c2-bb984a209c59.png)](https://user-images.githubusercontent.com/25839518/76718895-13d75300-6773-11ea-88c2-bb984a209c59.png)

[apply:](https://tc39.es/ecma262/#sec-function.prototype.apply)
[![image](https://user-images.githubusercontent.com/25839518/76718908-1b96f780-6773-11ea-8db6-37eea7cb1576.png)](https://user-images.githubusercontent.com/25839518/76718908-1b96f780-6773-11ea-8db6-37eea7cb1576.png)

可以看到算法步骤中，apply多了`CreateListFromArrayLike`的调用，其他的操作几乎是一样的（甚至apply仍然多了点操作）。从草案的算法描述来看，call性能 > apply性能。

