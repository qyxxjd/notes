
#### MVP
> View 接收用户交互请求  
> View 将请求转交给 Presenter(V调用P接口)  
> Presenter 操作Model进行数据更新(P调用M接口)  
> Model 通知Presenter数据发生变化(M调用P接口)  
> Presenter 更新View数据(P执行接口,V相应回调)  

- Presenter完全把Model和View进行了分离，主要的程序逻辑在Presenter里实现。
- Presenter与具体的View是没有只直接关联的，而是通过定义好的接口进行交互
- 缺点：Presenter比较重


#### MVVM
> View 接收用户交互请求  
> View 将请求转交给ViewModel  
> ViewModel 操作Model数据更新  
> Model 更新完数据，通知ViewModel数据发生变化  
> ViewModel 更新View数据  

优点：
- ViewModle易于单元测试
- 相同业务的ViewModel可以复用，官方最新的示例项目多使用这种方案
- 有类似Activity/Fragment的状态恢复机制


#### MVI
> 用户操作以Intent的形式通知Model  
> Model基于Intent更新State  
> View接收到State变化刷新UI  

- Model: 与MVVM中的Model不同的是，MVI的Model主要指UI状态（State）。例如页面加载状态、控件位置等都是一种UI状态
- View: 与其他MVX中的View一致，可能是一个Activity或者任意UI承载单元。MVI中的View通过订阅Model的变化实现界面刷新
- Intent: 此Intent不是Activity的Intent，用户的任何操作都被包装成Intent后发送给Model层进行数据请求
