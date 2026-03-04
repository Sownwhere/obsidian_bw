# 模型初始化方式对比：models[...] vs source

在多智能体强化学习框架中，我们经常会看到类似如下两种模型构造方式：

```python
models[agent_id][role] = model_class(...)
```

和

```python
source = model_class(..., return_source=True)
```

虽然它们表面上看起来类似，都是在构建模型实例，但在目的和使用场景上有明显区别。以下是详细对比分析：

---

## 1. 共同点

- 都在调用同一个模型类 `model_class(...)`。
    
- 都传入类似的参数，如 observation space、action space、device 以及模型结构配置。
    
- 都会返回一个模型实例。
    

---

## 2. 核心区别

|比较点|`models[agent_id][role] = model_class(...)`|`source = model_class(..., return_source=True)`|
|---|---|---|
|📌 作用目标|把模型注册/存储在框架内部结构中供训练使用|单独获取模型源对象，通常用于进一步处理或拷贝|
|🧠 目标对象|存入 `models` 字典，用于 agent 的策略或 critic 表示|获取“裸模型”，可能不直接注册|
|📦 返回值处理|不关心 wrapper，作为完整对象保存|返回模型的底层结构而非封装体|
|🛠 常用于|初始化阶段，配置 agent 所用的模型|临时构建模型，用于共享结构或模板|

---

## 3. return_source=True 的作用

一些框架（如 `skrl`）的模型类中，会使用包装器包裹住底层网络。此时：

- 默认返回的是带有 sampling/logging 等功能的包装对象。
    
- 若传入 `return_source=True`，则返回原始的底层网络（如 `nn.Module`）。
    

### 示例：

```python
class MyWrappedModel:
    def __init__(..., return_source=False):
        if return_source:
            return self.network  # 返回最底层网络结构
        else:
            return self          # 返回包装后的对象
```

---

## 4. 举个例子

```python
# 正式初始化 agent_0 的 policy 网络：
models["agent_0"]["policy"] = MLPPolicy(...)

# 临时构造一个结构相同的模型副本：
source = MLPPolicy(..., return_source=True)
```

---

## 5. 总结

- `models[agent_id][role] = model_class(...)`
    
    - 正式为 agent 初始化模型。
        
    - 存入系统结构中供训练使用。
        
- `source = model_class(..., return_source=True)`
    
    - 构造一个“裸模型”用于其他目的，如：拷贝、共享参数、分析。
        
    - 不一定注册到系统结构中。
        

这两种方式虽然底层调用相同的模型类，但服务于不同的使用流程。了解它们的区别有助于你更好地掌控强化学习框架的模型管理逻辑。