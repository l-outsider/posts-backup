# Page 1

## Flex 布局中的自动拉伸行为

### 默认行为 (`align-items: stretch`)

* 作用于 flex 容器的交叉轴
  * 如果是 `flex-direction: row`，交叉轴是垂直方向
  * 如果是 `flex-direction: column`，交叉轴是水平方向

### 子元素表现

* #### 未设置尺寸时
  * 自动拉伸填满容器的交叉轴
  * ```css
    .container {
      display: flex;
      height: 100px;
    }
    .item {
      /* 会自动拉伸到 100px 高度 */
    }
    ```
* #### 设置固定尺寸时
  * 保持设置的尺寸
  * 不会被拉伸
  * ```css
    .container {
      display: flex;
      height: 100px;
    }
    .item {
      height: 50px; /* 会保持 50px 高度 */
    }
    ```

### 覆盖默认行为
