### JavaScript Math 对象笔记

Math 是 JavaScript 内置的数学工具对象，提供**数学常数**和**数学函数**。所有属性和方法都是静态的，无需创建实例，直接通过 `Math.xxx` 调用。

---

#### 一、常用数学常数（属性）
| 属性          | 值                  | 描述                     |
|---------------|---------------------|--------------------------|
| `Math.PI`     | ≈ 3.141592653589793 | 圆周率 π                 |
| `Math.E`      | ≈ 2.718281828459045 | 自然对数的底数 e         |
| `Math.SQRT2`  | ≈ 1.414213562373095 | 2 的平方根               |
| `Math.SQRT1_2`| ≈ 0.707106781186547 | 1/2 的平方根             |
| `Math.LN2`    | ≈ 0.693147180559945 | 2 的自然对数             |
| `Math.LN10`   | ≈ 2.302585092994046 | 10 的自然对数            |
| `Math.LOG2E`  | ≈ 1.442695040888963 | 以 2 为底 e 的对数       |
| `Math.LOG10E` | ≈ 0.434294481903251 | 以 10 为底 e 的对数      |

```javascript
console.log(Math.PI); // 输出: 3.141592653589793
```

---

#### 二、常用数学函数（方法）

##### 1. 基本计算
| 方法                  | 示例                     | 描述                          |
|-----------------------|--------------------------|-------------------------------|
| `Math.abs(x)`         | `Math.abs(-5) // 5`      | 返回绝对值                    |
| `Math.sign(x)`        | `Math.sign(-10) // -1`   | 返回符号：正数=1, 负数=-1, 0=0 |
| `Math.sqrt(x)`        | `Math.sqrt(16) // 4`     | 返回平方根                    |
| `Math.cbrt(x)`        | `Math.cbrt(27) // 3`     | 返回立方根                    |
| `Math.pow(x, y)`      | `Math.pow(2, 3) // 8`    | 返回 x 的 y 次幂（等价于 `x**y`） |
| `Math.hypot(x1, x2,…)`| `Math.hypot(3,4) // 5`   | 返回所有参数平方和的平方根（勾股定理）|

---

##### 2. 取整方法
| 方法              | 示例                        | 描述                          |
|-------------------|-----------------------------|-------------------------------|
| `Math.floor(x)`   | `Math.floor(3.9) // 3`      | 向下取整（向小方向舍入）      |
| `Math.ceil(x)`    | `Math.ceil(3.1) // 4`       | 向上取整（向大方向舍入）      |
| `Math.round(x)`   | `Math.round(3.5) // 4`      | 四舍五入（.5 时向正无穷舍入） |
| `Math.trunc(x)`   | `Math.trunc(3.9) // 3`      | 直接去除小数部分              |
| `Math.fround(x)`  | `Math.fround(3.14159)`      | 返回最接近的单精度浮点数      |

```javascript
Math.round(-1.5); // -1 （注意：负数时 .5 向正无穷舍入）
```

---

##### 3. 最值方法
| 方法                     | 示例                          | 描述                  |
|--------------------------|-------------------------------|-----------------------|
| `Math.max(a, b, c, ...)` | `Math.max(1, 3, 2) // 3`      | 返回参数中的最大值    |
| `Math.min(a, b, c, ...)` | `Math.min(1, 3, 2) // 1`      | 返回参数中的最小值    |

```javascript
// 求数组最大值
const arr = [5, 2, 9];
Math.max(...arr); // 9
```

---

##### 4. 随机数
| 方法              | 描述                                  |
|-------------------|---------------------------------------|
| `Math.random()`   | 返回 [0, 1) 区间的随机浮点数          |

```javascript
// 生成 [min, max] 的随机整数
function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}
getRandomInt(1, 10); // 随机生成 1~10 的整数
```

---

##### 5. 三角函数（参数为弧度）
| 方法              | 描述         |
|-------------------|--------------|
| `Math.sin(x)`     | 正弦         |
| `Math.cos(x)`     | 余弦         |
| `Math.tan(x)`     | 正切         |
| `Math.asin(x)`    | 反正弦        |
| `Math.acos(x)`    | 反余弦        |
| `Math.atan(x)`    | 反正切        |
| `Math.atan2(y,x)` | 从 x 轴到点 (x,y) 的角度（-π 到 π） |

```javascript
// 角度转弧度：radians = (degrees * Math.PI) / 180
const sin30 = Math.sin(30 * Math.PI / 180); // 0.5
```

---

##### 6. 对数方法
| 方法              | 描述                  |
|-------------------|-----------------------|
| `Math.log(x)`     | 返回 x 的自然对数（ln(x)） |
| `Math.log10(x)`   | 返回以 10 为底的对数  |
| `Math.log2(x)`    | 返回以 2 为底的对数   |
| `Math.exp(x)`     | 返回 e^x 的值         |

---

#### 三、使用技巧与注意事项
1. **参数处理**：
   - 非数值参数会被自动转换为数值（如 `null→0`, `"5"→5`）。
   - 无效输入返回 `NaN`：`Math.sqrt(-1) // NaN`。

2. **性能优化**：
   - `Math.floor()` 比 `parseInt()` 更快。
   - 位运算取整：`x | 0` （仅适用于 32 位整数）。

3. **生成随机颜色**：
   ```javascript
   const randomColor = `#${Math.floor(Math.random() * 0xFFFFFF).toString(16).padStart(6, '0')}`;
   ```

4. **计算两点距离**：
   ```javascript
   const distance = Math.hypot(x2 - x1, y2 - y1);
   ```

---

#### 四、完整示例
```javascript
// 计算圆面积
const radius = 5;
const area = Math.PI * Math.pow(radius, 2); // 78.5398...

// 生成 [-10, 10] 的随机整数
const rand = Math.floor(Math.random() * 21) - 10;

// 角度转弧度函数
function toRadians(degrees) {
  return degrees * Math.PI / 180;
}
console.log(Math.sin(toRadians(90))); // 1
```

> 提示：Math 对象方法完整列表参考 [MDN Math 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Math)。