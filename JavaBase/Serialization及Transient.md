# Serialization及Transient



## Serialization

> 将对象的状态信息转换为可以存储或传输的形式
>
> 在序列化期间，对象将其当前状态写入到临时或者持久性存储区，以后，可以通过存储区中读取或反序列化对象状态，重新创建该对象



- serialVersionUID 流的唯一标识符(序列版本) 适用于java序列化机制。JAVA序列化的机制是通过**判断类的serialVersionUID来验证的版本一致的**。

  

- 在进行反序列化时，JVM会把传来的字节流中的serialVersionUID于本地相应实体类的serialVersionUID进行比较。**如果相同说明是一致的，可以进行反序列化**，否则会出现反序列化版本一致的异常，即是InvalidCastException。



## Transient

transient的作用就是把状态的生命周期仅存于调用者的内存中而不会写到磁盘里持久化



Java序列化提供两种方式。

- 一种是实现Serializable接口

- 另一种是实现Exteranlizable接口。 需要重写writeExternal和readExternal方法，它的效率比Serializable高一些，并且可以决定哪些属性需要序列化（即使是transient修饰的），但是对大量对象，或者重复对象，则效率低。

  > 所以被transient关键字修饰过得变量不一定不能被序列化



注意，

静态变量由于在全局区，不管是不是transient关键字修饰，都不会被序列化
