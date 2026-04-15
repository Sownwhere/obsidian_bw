# Jacobian 解析推导文档

## 1. 问题定义

给定函数：
$v = f(\text{imu}, \text{encoder})$

目标是计算Jacobian矩阵：
$J = \begin{bmatrix} \frac{\partial v}{\partial \text{imu}} & \frac{\partial v}{\partial \text{encoder}} \end{bmatrix}$

## 2. 函数结构分解

### 2.1 整体函数链

```
v = f(imu, encoder)
├── v = v_bar * cos(v_theta)
│   ├── v_bar = w_encoder * d_bar (常数)
│   └── v_theta = theta_2 - theta_1
│       ├── theta_2 = bar_theta - (imu - encoder)
│       └── theta_1 = arctan(d_2 / d_horizontal)
│           ├── d_2 = A * cos(imu)
│           │   └── A = d_vertical - d_1 * tan(imu) - d_bar * cos(bar_theta) / cos(imu)
│           └── d_horizontal = d_bar * sin(theta_2) + d_vertical * sin(imu) + d_1 * cos(imu)
```

## 3. 逐层求导推导

### 3.1 第一层：对 $v$ 求导

$\frac{\partial v}{\partial x} = \frac{\partial v_{bar}}{\partial x} \cdot \cos(v_\theta) + v_{bar} \cdot (-\sin(v_\theta)) \cdot \frac{\partial v_\theta}{\partial x}$

由于 $v_{bar}$ 是常数，$\frac{\partial v_{bar}}{\partial x} = 0$，因此：
$\frac{\partial v}{\partial x} = -v_{bar} \cdot \sin(v_\theta) \cdot \frac{\partial v_\theta}{\partial x}$

### 3.2 第二层：对 $v_\theta$ 求导

$v_\theta = \theta_2 - \theta_1$
$\frac{\partial v_\theta}{\partial x} = \frac{\partial \theta_2}{\partial x} - \frac{\partial \theta_1}{\partial x}$

### 3.3 第三层：对 $\theta_2$ 求导

$\theta_2 = \bar{\theta} - (\text{imu} - \text{encoder})$
$\frac{\partial \theta_2}{\partial \text{imu}} = -1, \quad \frac{\partial \theta_2}{\partial \text{encoder}} = +1$

### 3.4 第四层：对 $\theta_1$ 求导

$\theta_1 = \arctan\left(\frac{d_2}{d_h}\right)$

设 $u = \frac{d_2}{d_h}$，则：
$\frac{\partial \theta_1}{\partial x} = \frac{1}{1 + u^2} \cdot \frac{\partial u}{\partial x} = \frac{1}{1 + (d_2/d_h)^2} \cdot \frac{\partial}{\partial x}\left(\frac{d_2}{d_h}\right)$

分式求导：
$\frac{\partial}{\partial x}\left(\frac{d_2}{d_h}\right) = \frac{d_h \cdot \frac{\partial d_2}{\partial x} - d_2 \cdot \frac{\partial d_h}{\partial x}}{d_h^2}$

## 4. 关键变量导数计算

### 4.1 $d_{horizontal}$ 的偏导数

$d_h = d_{bar}\sin(\theta_2) + d_{vertical}\sin(\text{imu}) + d_1\cos(\text{imu})$

**对 imu 的偏导：**
$\frac{\partial d_h}{\partial \text{imu}} = d_{bar}\cos(\theta_2)\cdot \frac{\partial \theta_2}{\partial \text{imu}} + d_{vertical}\cos(\text{imu}) - d_1\sin(\text{imu})$

**对 encoder 的偏导：**
$\frac{\partial d_h}{\partial \text{encoder}} = d_{bar}\cos(\theta_2)\cdot \frac{\partial \theta_2}{\partial \text{encoder}}$

### 4.2 $d_2$ 的偏导数

$d_2 = A \cdot \cos(\text{imu})$
其中：
$A = d_{vertical} - d_1 \tan(\text{imu}) - \frac{d_{bar}\cos(\bar{\theta})}{\cos(\text{imu})}$

**乘积法则：**
$\frac{\partial d_2}{\partial x} = \frac{\partial A}{\partial x}\cos(\text{imu}) + A \cdot (-\sin(\text{imu})) \cdot \frac{\partial \text{imu}}{\partial x}$

**对 imu 的偏导：**
$\frac{\partial A}{\partial \text{imu}} = -d_1 \sec^2(\text{imu}) - d_{bar}\cos(\bar{\theta})\cdot \frac{\sin(\text{imu})}{\cos^2(\text{imu})}$

## 5. 完整的Jacobian计算流程

### 5.1 计算中间变量

1. 计算 $\theta_2 = \bar{\theta} - (\text{imu} - \text{encoder})$
2. 计算 $d_h = d_{bar}\sin(\theta_2) + d_{vertical}\sin(\text{imu}) + d_1\cos(\text{imu})$
3. 计算 $A = d_{vertical} - d_1 \tan(\text{imu}) - \frac{d_{bar}\cos(\bar{\theta})}{\cos(\text{imu})}$
4. 计算 $d_2 = A \cdot \cos(\text{imu})$
5. 计算 $\theta_1 = \arctan(d_2 / d_h)$
6. 计算 $v_\theta = \theta_2 - \theta_1$
7. 计算 $v = v_{bar} \cdot \cos(v_\theta)$

### 5.2 计算偏导数

**对 imu 的偏导：**

1. $\frac{\partial \theta_2}{\partial \text{imu}} = -1$
2. $\frac{\partial d_h}{\partial \text{imu}} = d_{bar}\cos(\theta_2)\cdot(-1) + d_{vertical}\cos(\text{imu}) - d_1\sin(\text{imu})$
3. $\frac{\partial A}{\partial \text{imu}} = -d_1 \sec^2(\text{imu}) - d_{bar}\cos(\bar{\theta})\cdot \frac{\sin(\text{imu})}{\cos^2(\text{imu})}$
4. $\frac{\partial d_2}{\partial \text{imu}} = \frac{\partial A}{\partial \text{imu}}\cos(\text{imu}) - A\sin(\text{imu})$
5. $\frac{\partial u}{\partial \text{imu}} = \frac{d_h \cdot \frac{\partial d_2}{\partial \text{imu}} - d_2 \cdot \frac{\partial d_h}{\partial \text{imu}}}{d_h^2}$
6. $\frac{\partial \theta_1}{\partial \text{imu}} = \frac{1}{1 + (d_2/d_h)^2} \cdot \frac{\partial u}{\partial \text{imu}}$
7. $\frac{\partial v_\theta}{\partial \text{imu}} = \frac{\partial \theta_2}{\partial \text{imu}} - \frac{\partial \theta_1}{\partial \text{imu}}$
8. $\frac{\partial v}{\partial \text{imu}} = -v_{bar} \cdot \sin(v_\theta) \cdot \frac{\partial v_\theta}{\partial \text{imu}}$

**对 encoder 的偏导：**

1. $\frac{\partial \theta_2}{\partial \text{encoder}} = +1$
2. $\frac{\partial d_h}{\partial \text{encoder}} = d_{bar}\cos(\theta_2)\cdot(+1)$
3. $\frac{\partial A}{\partial \text{encoder}} = 0$ (A不直接依赖encoder)
4. $\frac{\partial d_2}{\partial \text{encoder}} = 0$ (d_2不直接依赖encoder)
5. $\frac{\partial u}{\partial \text{encoder}} = \frac{-d_2 \cdot \frac{\partial d_h}{\partial \text{encoder}}}{d_h^2}$
6. $\frac{\partial \theta_1}{\partial \text{encoder}} = \frac{1}{1 + (d_2/d_h)^2} \cdot \frac{\partial u}{\partial \text{encoder}}$
7. $\frac{\partial v_\theta}{\partial \text{encoder}} = \frac{\partial \theta_2}{\partial \text{encoder}} - \frac{\partial \theta_1}{\partial \text{encoder}}$
8. $\frac{\partial v}{\partial \text{encoder}} = -v_{bar} \cdot \sin(v_\theta) \cdot \frac{\partial v_\theta}{\partial \text{encoder}}$

## 6. 实现建议

### 6.1 代码结构优化

```
def compute_jacobian(imu, encoder, params):
    # 计算中间变量
    theta_2 = params['bar_theta'] - (imu - encoder)
    d_h = (params['d_bar'] * np.sin(theta_2) + 
           params['d_vertical'] * np.sin(imu) + 
           params['d_1'] * np.cos(imu))
    
    A = (params['d_vertical'] - params['d_1'] * np.tan(imu) - 
         params['d_bar'] * np.cos(params['bar_theta']) / np.cos(imu))
    d_2 = A * np.cos(imu)
    
    theta_1 = np.arctan2(d_2, d_h)
    v_theta = theta_2 - theta_1
    v = params['v_bar'] * np.cos(v_theta)
    
    # 计算偏导数
    # 对imu的偏导
    d_theta2_d_imu = -1.0
    
    d_dh_d_imu = (params['d_bar'] * np.cos(theta_2) * d_theta2_d_imu + 
                  params['d_vertical'] * np.cos(imu) - 
                  params['d_1'] * np.sin(imu))
    
    dA_d_imu = (-params['d_1'] / np.cos(imu)**2 - 
                params['d_bar'] * np.cos(params['bar_theta']) * 
                np.sin(imu) / np.cos(imu)**2)
    
    d_d2_d_imu = dA_d_imu * np.cos(imu) - A * np.sin(imu)
    
    u = d_2 / d_h
    d_u_d_imu = (d_h * d_d2_d_imu - d_2 * d_dh_d_imu) / (d_h**2)
    d_theta1_d_imu = (1 / (1 + u**2)) * d_u_d_imu
    
    d_vtheta_d_imu = d_theta2_d_imu - d_theta1_d_imu
    d_v_d_imu = -params['v_bar'] * np.sin(v_theta) * d_vtheta_d_imu
    
    # 对encoder的偏导
    d_theta2_d_enc = 1.0
    d_dh_d_enc = params['d_bar'] * np.cos(theta_2) * d_theta2_d_enc
    
    d_u_d_enc = (-d_2 * d_dh_d_enc) / (d_h**2)
    d_theta1_d_enc = (1 / (1 + u**2)) * d_u_d_enc
    
    d_vtheta_d_enc = d_theta2_d_enc - d_theta1_d_enc
    d_v_d_enc = -params['v_bar'] * np.sin(v_theta) * d_vtheta_d_enc
    
    return np.array([d_v_d_imu, d_v_d_enc]), v
```

### 6.2 数值验证

建议使用数值微分进行验证：

```
def numerical_jacobian(func, x, epsilon=1e-6):
    n = len(x)
    jac = np.zeros(n)
    f0 = func(x)
    
    for i in range(n):
        x_plus = x.copy()
        x_plus[i] += epsilon
        f_plus = func(x_plus)
        
        jac[i] = (f_plus - f0) / epsilon
    
    return jac
```

## 7. 注意事项

1. **三角函数单位**：确保所有角度使用弧度制
2. **数值稳定性**：注意 $\cos(\text{imu}) \approx 0$ 时的数值问题
3. **边界条件**：检查 $d_h \approx 0$ 时的除零错误
4. **精度控制**：选择合适的数值微分步长进行验证

通过以上完整的推导和实现方案，您可以准确计算所需的Jacobian矩阵。

