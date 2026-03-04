

## 1 文章概述

本文提出 **Code as Policies (CaP)**，一种将大型语言模型（LLMs）用于机器人控制的新方法。核心思想是：

- 利用 **代码生成 LLM**（例如 OpenAI Codex）将自然语言指令转换为 **可执行机器人策略代码**。
- 生成的代码被称为 **Language Model Programs (LMPs)**，可直接在真实机器人上运行。
- LMP 可以表达 **感知-控制闭环逻辑**，将高层语言描述映射为低层控制动作。

CaP 的目标不仅是生成动作序列，更是生成**可复用、可层级化、可解释的程序**，从而提高机器人策略的灵活性和通用性。

---

## 2 CaP 能实现的任务
1. **自然语言指令到动作映射**
   - 例如：
     - “把左边面积大于0.2的 block 移动到红碗右边”
     - “把蓝色 block 放到同色碗里”
   - LMP 会生成相应 Python 代码来执行动作。

2. **复杂策略生成**
   - 支持循环、条件判断、嵌套函数调用，实现多目标操作。
   - 例如根据对象位置和大小动态调整动作。

3. **感知驱动控制**
   - 利用视觉模型（如 ViLD、MDETR）获取物体位置和 bounding box。
   - 感知结果作为控制 API 输入参数，实现闭环反馈。

4. **泛化能力**
   - 能处理新对象、新环境和未见过的指令。
   - 结合 open-vocabulary perception，无需额外训练数据。

5. **层级函数生成与复用**
   - LMP 可以递归生成未定义函数，并在后续 LMP 中复用。
   - 保证策略逻辑清晰、模块化。

6. **高可解释性**
   - 生成的策略以代码形式呈现，便于理解、修改和复用。
   - 可直接调试或嵌入人机交互系统。

---

## 3. 方法概述：Code as Policies (CaP) 及 Language Model Programs (LMPs)

### 3.1 核心概念

- **LMP（Language Model Program）**：指由大语言模型（LLM）生成并可在系统上执行的程序。
    
- **CaP（Code as Policies）**：将自然语言指令映射为可执行代码的 LMP，能够：
    
    1. 对感知输入（传感器数据或高级感知模块）作出反应；
        
    2. 参数化低级控制 API（如抓取、移动、导航）；
        
    3. 直接在机器人系统上编译执行，实现具体任务。
        

**示例**：

```python
# stack the blocks in the empty bowl
empty_bowl_name = parse_obj('empty bowl')
block_names = parse_obj('blocks')
obj_names = [empty_bowl_name] + block_names
stack_objs_in_order(obj_names=obj_names)
```

- `parse_obj` 用于将自然语言对象描述解析为具体对象名称。
    
- `stack_objs_in_order` 是由 LMP 定义的函数，利用现有控制原语 `put_first_on_second` 实现堆叠动作。
    

---

### 3.2 LMP 的执行流程

1. **安全检查**：
    
    - 禁止 import 语句、`__` 开头的特殊变量，以及 `exec` / `eval` 调用。
        
2. **执行方式**：
    
    - 使用 Python 的 `exec` 函数执行 LMP 生成的代码。
        
    - 通过 `globals` 提供所有可调用 API（控制和感知函数）。
        
    - `locals` 为空字典，用于保存 LMP 中定义的新变量和函数。
        
3. **返回值**：
    
    - 如果 LMP 需要返回结果，从 `locals` 中读取。
        

---

### 3.3 Prompt 设计

LMP 的生成依赖于 **Hints** 和 **Examples**：

1. **Hints**：
    
    - 导入语句，告知模型可调用的 API。
        
    - 类型提示或变量命名示例，让 LLM 理解数据结构。
        
    
    ```python
    import numpy as np
    from utils import get_obj_names, put_first_on_second
    ```
    
2. **Examples**：
    
    - 少量的自然语言指令 → 代码示例，教模型如何将指令转为 Python 代码。
        
    - 可以涉及算术、控制 API 调用、列表或向量操作等。
        
3. **增量提示**：
    
    - 可维护 LMP “session”，新指令可以引用之前的指令或动作（如“undo the last action”）。
        

---

### 3.4 LMP 的层级与组合能力

- **低级示例**：
    
    - 对点或对象位置进行算术操作：
        
    
    ```python
    ret_val = pts_np + [0.3, 0]  # 将所有点向右移动
    ret_val = pts_np[np.argmin(pts_np[:, 0]), :]  # 找到最左侧的点
    ```
    
- **控制流与循环**：
    
    - 可以使用 `while`、`for` 等循环构建反馈控制策略：
        
    
    ```python
    while get_pos('red block')[0] < get_pos('blue bowl')[0]:
        target_pos = get_pos('red block') + [0.05, 0]
        put_first_on_second('red block', target_pos)
    ```
    
- **函数嵌套与重用**：
    
    - 高级 LMP 可调用其他 LMP 或已知函数，逐步构建复杂策略：
        
    
    ```python
    block_name = parse_obj('the left most block')
    while block_name == 'red block':
        target_pos = get_pos(block_name) + [0.3, 0]
        put_first_on_second(block_name, target_pos)
        block_name = parse_obj('the left most block')
    ```
    

---

### 3.5 层级函数生成（Hierarchical Function Generation）

- LMP 可以生成尚未定义的函数，实现递归或分层逻辑：
    

```python
# 根据边界框面积筛选对象
def get_objs_bigger_than_area_th(obj_names, bbox_area_th):
    return [name for name in obj_names if get_obj_bbox_area(name) > bbox_area_th]

# 计算对象边界框面积
def get_obj_bbox_area(obj_name):
    x1, y1, x2, y2 = get_obj_bbox_xyxy(obj_name)
    return (x2 - x1) * (y2 - y1)
```

- **实现方式**：
    
    1. 解析 LMP 的抽象语法树（AST），找出未定义函数。
        
    2. 调用专门生成函数的 LMP 生成函数实现。
        
    3. 将函数加入执行范围，并递归生成子函数（Depth-first）。
        

---

### 3.6 感知到控制的闭环策略

- LMP 可以将感知输出（状态）映射到控制动作（API 调用），形成可编程反馈策略：
    
    - 输入：自然语言指令 + 感知数据（对象位置、边界框、特征等）
        
    - 输出：控制动作（抓取、移动、导航等）
        
- **可扩展性**：
    
    - 支持不同机器人系统、不同控制 API
        
    - 支持历史上下文、复杂逻辑、几何与空间推理
        

---


### 3.7 代码生成流程

1. **解析自然语言指令**
   - 将语言描述转为逻辑操作、对象选择或动作参数
   - LLM 可理解抽象描述（颜色、方位、大小等）

2. **生成 LMP**
   - 低层 LMP：直接调用控制 API
   - 高层 LMP：控制流 + 层级函数调用 + 感知逻辑
   - 支持递归函数生成和组合已有 LMP

3. **递归函数生成**
   - 解析生成代码的 AST，寻找未定义函数
   - 调用函数生成 LMP 来实现这些函数
   - 持续递归直到所有函数定义完成

4. **执行代码**
   - 安全检查（禁止 exec/eval、禁止使用 `__` 特殊变量）
   - 使用 Python `exec` 执行 LMP
   - `globals` 提供所有 API，`locals` 存储生成变量和函数
   - 如果 LMP 需要返回值，则从 `locals` 获取

5. **闭环控制**
   - 使用 while/for 循环实现反馈控制
   - 感知结果驱动低层动作
   - 可对动作大小、速度或方向进行动态调整

---


## 4 CaP 实验与任务展示


### 4.1 层级 LMPs 在代码生成基准上的表现（Hierarchical LMPs on Code-Generation Benchmarks）

- **评估目标**：测试 CaP 的层级代码生成（Hierarchical Code Generation）方法在不同类型的代码生成任务中的效果，以及其泛化能力。
    
- **使用的基准**：
    
    1. **RoboCodeGen**：自建的机器人主题代码生成基准，包含 37 个函数生成问题，特点如下：
        
        - 机器人相关：空间推理（如找到一组点中最近的点）、几何判断（如检查一个边界框是否包含另一个）、控制问题（如 PD 控制）。
            
        - 支持使用第三方库（如 NumPy）来简化复杂运算。
            
        - 提供的函数头部没有文档字符串或显式类型提示，模型需自行推断。
            
        - 允许使用尚未定义的函数，并通过层级代码生成生成这些函数。
            
    2. **HumanEval**：标准通用代码生成问题，用于评估模型通用编程能力。
        
- **实验方法**：
    
    - 选用 4 个 OpenAI LLM 模型（Codex 及 GPT-3 系列）进行评测。
        
    - 使用标准度量：生成代码通过人工编写的单元测试的百分比（pass rate）。
        
- **关键发现**：
    
    1. **层级生成优于平铺生成**：
        
        - 层级生成允许模型将复杂函数拆解为多个子函数，分别生成并组合，提升了代码正确性和可读性。
            
        - 在所有模型和任务上，层级生成的通过率均高于平铺生成。
            
    2. **模型规模影响性能**：
        
        - 更大的模型（如 Codex davinci 175B）在层级生成任务上表现更佳。
            
        - 小模型（如 cushman）未必能充分利用层级生成的优势，说明代码生成能力达到一定水平后，层级生成才能发挥效果。
            
    3. **泛化能力提升**：
        
        - 层级生成在需要生成更长代码或多层逻辑的新指令（Productivity 类型泛化）上提升最明显。
            
        - 在 HumanEval 基准中，层级生成也能提升通用代码生成能力，表明不仅限于机器人任务。
            
- **总结**：
    
    - 层级 LMPs 通过拆分、复用和递归函数生成，提高了复杂策略生成和通用编程任务的准确性。
        
    - 对复杂、多步骤机器人操作任务尤其有效，因为这些任务通常需要多层逻辑和函数调用。
        


---

### 4.2 绘制形状（Waypoints）

- **任务描述**：UR5e 机械臂根据语言指令生成二维轨迹并绘制形状，如三角形、正方形、字母等。
    
- **方法**：
    
    1. LMP 解析自然语言指令生成 **waypoints**。
        
    2. 调用机械臂 API 顺序移动末端执行器到每个 waypoint。
        
- **示例代码**：
    

```python
# 画一个三角形
waypoints = parse_waypoints('draw a triangle')
for pt in waypoints:
    robot.move_to(pt)
```

- **能力展示**：
    
    - 可解析指令中的精确尺寸、形状、方向。
        
    - 支持历史轨迹操作（如“在已画形状的基础上添加新的边”）。
        
    - LMP 可以组合控制逻辑、闭环检测和函数复用生成复杂图形。
        

---

### 4.3 桌面物体抓取与搬运（Pick & Place）

- **实验环境**：UR5e + 吸盘抓手 + Realsense D435 相机
    
- **任务示例**：
    
    - 根据颜色、形状、位置、空间关系搬运物体。
        
    - 例如将蓝色方块放入蓝色碗，将红色方块放入红色碗。
        
- **示例代码**：
    

```python
put_first_on_second('blue block', 'blue bowl')
put_first_on_second('red block', 'red bowl')
```

- **高级功能**：
    
    - **历史操作回滚**：支持“undo last action”。
        
    - **空间几何推理**：根据物体大小、位置关系执行操作，例如“把最小的方块放到右边”。
        
    - 支持循环控制和条件判断，实现闭环操作。
        

---

### 4.4 桌面物体操作模拟评估（Simulation）

- **实验设置**：UR5e + Robotiq 2F85，操作 10 个彩色方块和 10 个彩色碗。
    
- **任务类型**：
    
    - **长序列任务**：多步操作，例如“把所有方块放入对应颜色碗中”。
        
    - **精细空间布局任务**：如“将方块排成对角线”。
        
- **实验对比**：
    
    - **CLIPort（监督学习）**：训练 30k 演示数据。
        
    - **LLM Planner（自然语言）**：直接用 LLM 解析指令生成动作。
        
- **结果**：
    
    - CaP 在未见任务属性时保持高性能，而 CLIPort 泛化能力下降。
        
    - CaP 能够执行需要精确数值计算和空间几何推理的任务，这些是自然语言 planner 难以实现的。
        
- **能力展示**：
    
    - 多任务组合、属性筛选（颜色、大小）、空间关系推理。
        
    - 高度灵活，少量示例即可完成未见任务。
        

---

### 4.5 移动机器人导航与操作（Mobile Robot Navigation & Manipulation）

- **实验环境**：移动机器人 + 7 自由度机械臂，厨房环境。
    
- **感知**：对象检测 APIs via ViLD [3]
    
- **动作**：导航、抓取、放置动作，支持绝对坐标和对象名操作。
    
- **示例代码**：
    

```python
# 拿起桌上红色杯子，放到左边台面
robot.navigate_to('red cup')
robot.pick('red cup')
robot.place([x_target, y_target, z_target])
```

- **能力展示**：
    
    - 支持长序列操作，处理复杂空间几何关系。
        
    - 可将高层任务拆解为导航 + 抓取 + 放置子任务。
        
    - 跨机器人系统部署，体现灵活性与易用性。
        

---


### 4.5 示例分析

#### 基础示例

```python
# move the purple bowl to the left
target_pos = get_pos('purple bowl') + [-0.3, 0]
put_first_on_second('purple bowl', target_pos)

# move the red block a bit to the right
target_pos = get_pos('red block') + [0.1, 0]
put_first_on_second('red block', target_pos)

# put the blue block on the same color bowl
put_first_on_second('blue block', 'blue bowl')

````

- LMP 能理解语言描述：“a bit”表示微调位置，“same color”表示颜色匹配。
    
- 通过 get_pos 获取对象位置，再调用 put_first_on_second 执行动作。
    

#### 高级示例：反馈循环 + 层级函数

```python
# move blocks bigger than 0.2 area left of red bowl to the right
block_names = parse_obj('blocks with area bigger than 0.2 that are left of the red bowl')

while len(block_names) > 0:
    for block_name in block_names:
        target_pos = get_pos(block_name) + np.array([0.1, 0])
        put_first_on_second(block_name, target_pos)
    block_names = parse_obj('blocks with area bigger than 0.2 that are left of the red bowl')
```

- `parse_obj` 是子 LMP，用于将语言描述解析为对象列表。
    
- 支持循环和条件判断，实现闭环策略。
    
- 高层 LMP 调用低层 LMP，实现模块化、层级化策略生成。
    

#### 函数层级生成示例

```python
# define function: get_objs_bigger_than_area_th
def get_objs_bigger_than_area_th(obj_names, bbox_area_th):
    return [name for name in obj_names if get_obj_bbox_area(name) > bbox_area_th]

# define function: get_obj_bbox_area
def get_obj_bbox_area(obj_name):
    x1, y1, x2, y2 = get_obj_bbox_xyxy(obj_name)
    return (x2 - x1) * (y2 - y1)
```

- 未定义函数可自动由 LMP 生成。
    
- 支持递归函数定义，保证高层策略可复用。
    

---

#### 其他示例演示

1. **自然语言指令到动作映射**
    
    - LMP 可以将抽象的语言描述直接映射为具体的机器人动作。
        
    - **示例：**
        
        - 指令：“把左边面积大于0.2的 block 移动到红碗右边”
            
```python
block_names = parse_obj('blocks with area bigger than 0.2 that are left of the red bowl')
	while len(block_names) > 0:
		for block_name in block_names:
			target_pos = get_pos(block_name) + np.array([0.1, 0])
			put_first_on_second(block_name, target_pos)
		block_names = parse_obj('blocks with area bigger than 0.2 that are left of the red bowl')
```
            
        - LMP 会理解语言描述的逻辑：“左边”“面积大于0.2”“移动到红碗右边”，并生成闭环动作。
            
2. **颜色或属性关联**
    
    - 可以根据对象的属性（如颜色、类别）选择动作对象。
        
    - **示例：**
        
        - 指令：“把蓝色 block 放到同色碗里”
            
```python
put_first_on_second('blue block', 'blue bowl')
```
            
        - LMP 能理解“同色”这种抽象描述，自动匹配对象。
            
3. **闭环控制与反馈循环**
    
    - LMP 支持 while/for 循环，实现感知驱动的闭环控制。
        
    - **示例：**
        
        - 指令：“把红色 block 移动到蓝碗右边，每次移动 5cm，直到位置合适”
            
 ```python
while get_pos('red block')[0] < get_pos('blue bowl')[0]:
	target_pos = get_pos('red block') + [0.05, 0]
	put_first_on_second('red block', target_pos)
```
            
        - 机器人会实时读取物体位置，动态调整动作，直到任务完成。
            
4. **复杂策略组合**
    
    - 可同时处理多个对象、多条件操作。
        
    - **示例：**
        
        - 指令：“把面积大于0.2且在红碗左边的所有 block 移动到右边”
            
```python
block_names = parse_obj('blocks with area bigger than 0.2 that are left of the red bowl')
for block_name in block_names:
	target_pos = get_pos(block_name) + [0.1, 0]
	put_first_on_second(block_name, target_pos)
```
            
        - LMP 可以组合对象筛选、控制流和动作执行，实现多对象操作。
            
5. **抽象描述映射**
    
    - 可以将自然语言中不精确或抽象的描述映射为具体数值或动作。
        
    - **示例：**
        
        - 指令：“把碗向左移动一点”
            
```python
target_pos = get_pos('purple bowl') + [-0.3, 0]  # '一点'被映射为0.3单位
put_first_on_second('purple bowl', target_pos)
```
            
        - LMP 结合上下文和常识，将“a bit”转化为合理的数值。
            
6. **层级函数生成与复用**
    
    - 高层 LMP 可以调用低层 LMP 或生成新的函数，实现策略模块化。
        
    - **示例：**
        
        - 高层指令：“获取面积大于阈值的 block，并移动它们”
            
```python
		def get_objs_bigger_than_area_th(obj_names, bbox_area_th):
	return [name for name in obj_names if get_obj_bbox_area(name) > bbox_area_th]

block_names = get_objs_bigger_than_area_th(all_blocks, 0.2)
for block_name in block_names:
	target_pos = get_pos(block_name) + [0.1, 0]
	put_first_on_second(block_name, target_pos)
```
            
        - LMP 可自动生成 `get_objs_bigger_than_area_th`，并在后续策略中复用。
            
7. **多任务/多目标操作**
    
    - LMP 可以同时处理排序、堆叠、搬运等多种任务。
        
    - **示例：**
        
        - 指令：“先把蓝色 block 放到蓝碗里，再把红色 block 放到红碗里”
            
```python
put_first_on_second('blue block', 'blue bowl')
put_first_on_second('red block', 'red bowl')
```
            
        - 支持按顺序执行多步任务。
            
## 5 CaP 的优势

1. **自然语言驱动**
    
    - 用户无需编程即可控制机器人
        
    - 可理解抽象描述和常识指令
        
2. **任务适应性**
    
    - 能处理未见过的对象、指令和环境
        
    - 无需额外数据收集或训练
        
3. **可复用和可解释**
    
    - 策略以代码形式呈现
        
    - 支持调试和二次开发
        
4. **函数层级生成**
    
    - 自动生成缺失函数
        
    - 高层逻辑和低层动作分离
        
5. **闭环控制**
    
    - 感知结果驱动低层动作
        
    - 支持 while/for 循环，实现反馈和调整
        
6. **继承 LLM 优势**
    
    - 上下文理解、常识推理、多语言支持
        
    - 支持人机交互和对话式指令
        

---


---
