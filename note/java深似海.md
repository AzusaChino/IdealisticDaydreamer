# JAVA

## java面向过程编程

- 核心要素
  - 数据结构: 原生类型, 对象类型, 数组类型, 集合类型
  - 方法调用: 访问性, 返回类型, 方法参数, 异常等
  - 执行流程: 赋值, 逻辑, 迭代(循环), 递归等

## Java面向对象编程

(java9之前)

- public: all
- protected: 继承 + 同包
- (default): 同包
- private: 当前类

---

- 面向对象基本特性
  - 封装性
  - 继承性
  - 多态性
- 面向对象设计模式
  - GoF 23: 构建, 结构, 行为
  - 方法设计: 名称,访问性, 参数, 返回类型, 异常
  - 泛型设计: 类级别, 方法级别
  - 异常设计: 层次性, 传播性

---

- 单元：一个类或者一组类（组件）
  - 类采用名词结构
    - 动词过去式+名词
      - ContextRefreshedEvent
    - 动词ing + 名词
      - InitializingBean
    - 形容词 + 名词
      - ConfigurableApplicationContext
- 执行：某个方法
  - 方法命名：动词
    - execute
    - callback
    - run
  - 方法参数：名词
  - 异常：
    - 根（顶层）异常
      - Throwable
        - checked 类型：Exception
        - unchecked类型：RuntimeException
        - 不常见：Error
    - Java 1.4 `java.lang.StackTraceElement`
      - 添加异常原因（cause）
        - 反模式：吞掉某个异常
        - 性能：注意 `fillInStackTrace()` 方法的开销，避免异常栈调用深度
          - 方法一：JVM 参数控制栈深度（物理屏蔽）
          - 方法二：logback 日志框架控制堆栈输出深度（逻辑屏蔽）

## Java 函数式基础

- 面向函数编程(Since Java 8)
  - Lambda 表达式
  - 默认方法
  - 方法引用
- 匿名内置类
  - 典型场景
    - Java Event/Listener
    - Java Concurrent
    - Spring Template
  - 基本特点
    - 基于多态(多数基于接口编程)
    - 实现类无需名称
    - 允许多个抽象方法
  - 局限性
    - 代码臃肿
    - 强类型约束
    - 接口方法升级
- Lambda表达式
  - 基本特点
    - 流程编排清晰
    - 函数类型编程
    - 改善代码臃肿
    - 兼容接口升级
  - 实现手段
    - @FunctionInterface接口
    - Lambda语法
    - 方法引用
    - 接口default方法实现
  - 局限性
    - 单一抽象方法
    - Lambda调试困难
    - Stream API操作能力有限

## Java模块化基础

- Java 9 模块化
  - 收益
    - 提升平台伸缩性
    - 提升平台完整性
    - 提升性能
  - 定义模块
    - 模块声明
    - 模块依赖
    - 模块导出
  - 模块结构
    - 模块描述文件
    - 平台模块
    - 模块 artifacts

## Java 接口设计

## Java枚举设计
