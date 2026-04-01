# ✅ 最佳实践（Best Practices）

> 本章目标：掌握 Recharts 在实际项目中的使用技巧、性能优化方法和常见问题解决方案

---

## 🚀 性能优化（Performance Optimization）

### 1. 使用 ResponsiveContainer 的注意事项

```tsx
// ✅ 正确：父容器有明确尺寸
<div style={{ width: '100%', height: 400 }}>
  <ResponsiveContainer>
    <LineChart data={data}>...</LineChart>
  </ResponsiveContainer>
</div>

// ❌ 错误：父容器无高度 → ResponsiveContainer 高度为 0
<div>
  <ResponsiveContainer>
    <LineChart data={data}>...</LineChart>
  </ResponsiveContainer>
</div>
```

> 💡 `ResponsiveContainer` 使用 `ResizeObserver` 监测容器大小变化。确保父元素有明确的宽度和高度。

### 2. 大数据集优化

当数据点超过 1000 个时，性能可能成为问题：

```tsx
// ✅ 方法一：关闭动画
<Line dataKey="value" isAnimationActive={false} />

// ✅ 方法二：减少渲染的点
<Line dataKey="value" dot={false} />  {/* 不渲染数据点标记 */}

// ✅ 方法三：使用 Brush 实现视窗内数据渲染
<LineChart data={largeData}>
  <Line dataKey="value" />
  <Brush dataKey="name" height={30} />  {/* 用户可以选择查看的数据范围 */}
</LineChart>
```

### 3. 避免不必要的重新渲染

```tsx
// ❌ 错误：每次渲染都创建新的 data 数组
function MyChart() {
  const data = generateData(); // 每次渲染都生成新数组
  return <LineChart data={data}>...</LineChart>;
}

// ✅ 正确：使用 useMemo 缓存数据
function MyChart() {
  const data = useMemo(() => generateData(), [/* dependencies */]);
  return <LineChart data={data}>...</LineChart>;
}
```

```tsx
// ❌ 错误：内联对象导致不必要渲染
<Line dot={{ r: 5, fill: 'red' }} />  {/* 每次渲染创建新对象 */}

// ✅ 正确：提取为常量
const dotStyle = { r: 5, fill: 'red' };
<Line dot={dotStyle} />
```

### 4. 条件渲染优化

```tsx
// ✅ 利用 React 的条件渲染按需显示组件
<LineChart data={data}>
  <XAxis dataKey="name" />
  <YAxis />
  {showGrid && <CartesianGrid strokeDasharray="3 3" />}
  {showTooltip && <Tooltip />}
  {showLegend && <Legend />}
  <Line dataKey="value" />
</LineChart>
```

---

## 🎨 常见使用场景

### 场景 1：多 Y 轴图表

当两组数据的量级差异很大时，需要双 Y 轴：

```tsx
const data = [
  { month: '1月', revenue: 4000, growthRate: 0.12 },
  { month: '2月', revenue: 3000, growthRate: 0.08 },
  { month: '3月', revenue: 5000, growthRate: 0.15 },
];

<ComposedChart width={700} height={400} data={data}>
  <XAxis dataKey="month" />

  {/* 左侧 Y 轴：收入（大数值） */}
  <YAxis yAxisId="left" orientation="left" stroke="#8884d8" />

  {/* 右侧 Y 轴：增长率（小数值） */}
  <YAxis yAxisId="right" orientation="right" stroke="#82ca9d"
         tickFormatter={(value) => `${(value * 100).toFixed(0)}%`} />

  <Tooltip />
  <Legend />

  <Bar dataKey="revenue" yAxisId="left" fill="#8884d8" name="收入" />
  <Line dataKey="growthRate" yAxisId="right" stroke="#82ca9d" name="增长率" />
</ComposedChart>
```

> 💡 **关键**：使用 `yAxisId` 将数据系列绑定到对应的 Y 轴。

### 场景 2：自定义 Tooltip

```tsx
const CustomTooltip = ({ active, payload, label }) => {
  if (!active || !payload?.length) return null;

  return (
    <div style={{
      background: '#fff',
      padding: '12px 16px',
      border: '1px solid #e0e0e0',
      borderRadius: 8,
      boxShadow: '0 2px 8px rgba(0,0,0,0.1)',
    }}>
      <p style={{ margin: 0, fontWeight: 600, marginBottom: 8 }}>{label}</p>
      {payload.map((entry, index) => (
        <div key={index} style={{ display: 'flex', alignItems: 'center', gap: 8 }}>
          <span style={{
            width: 12, height: 12, borderRadius: '50%',
            background: entry.color, display: 'inline-block'
          }} />
          <span>{entry.name}：</span>
          <span style={{ fontWeight: 600 }}>{entry.value.toLocaleString()}</span>
        </div>
      ))}
    </div>
  );
};

<Tooltip content={<CustomTooltip />} />
```

### 场景 3：点击图表获取数据

```tsx
function InteractiveChart() {
  const [selectedData, setSelectedData] = useState(null);

  const handleClick = (data) => {
    if (data?.activePayload) {
      setSelectedData(data.activePayload[0].payload);
    }
  };

  return (
    <div>
      <BarChart width={600} height={300} data={data} onClick={handleClick}>
        <XAxis dataKey="name" />
        <YAxis />
        <Bar dataKey="value" fill="#8884d8" />
      </BarChart>

      {selectedData && (
        <div>
          选中了：{selectedData.name}，值为 {selectedData.value}
        </div>
      )}
    </div>
  );
}
```

### 场景 4：动态更新数据

```tsx
function RealTimeChart() {
  const [data, setData] = useState(initialData);

  useEffect(() => {
    const interval = setInterval(() => {
      setData(prevData => {
        const newPoint = {
          time: new Date().toLocaleTimeString(),
          value: Math.random() * 100,
        };
        // 保持最近 20 个数据点
        return [...prevData.slice(-19), newPoint];
      });
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  return (
    <LineChart width={600} height={300} data={data}>
      <XAxis dataKey="time" />
      <YAxis domain={[0, 100]} />
      <Line type="monotone" dataKey="value" stroke="#8884d8"
            isAnimationActive={false} />  {/* 实时数据关闭动画 */}
    </LineChart>
  );
}
```

### 场景 5：使用 Recharts Hooks

Recharts v3 提供了一组公共 Hooks，让你在自定义组件中访问图表状态：

```tsx
import { useChartWidth, useChartHeight, useActiveTooltipLabel } from 'recharts';

function CustomOverlay() {
  const chartWidth = useChartWidth();
  const chartHeight = useChartHeight();
  const activeLabel = useActiveTooltipLabel();

  return (
    <div style={{ position: 'absolute', top: 10, right: 10 }}>
      图表尺寸：{chartWidth} × {chartHeight}
      {activeLabel && <div>当前高亮：{activeLabel}</div>}
    </div>
  );
}
```

### 场景 6：图表同步

```tsx
function Dashboard() {
  return (
    <div>
      {/* 两个图表使用相同的 syncId，鼠标悬浮一个，另一个同步 */}
      <LineChart syncId="metrics" data={revenueData}>
        <XAxis dataKey="month" />
        <YAxis />
        <Tooltip />
        <Line dataKey="revenue" stroke="#8884d8" />
      </LineChart>

      <BarChart syncId="metrics" data={costData}>
        <XAxis dataKey="month" />
        <YAxis />
        <Tooltip />
        <Bar dataKey="cost" fill="#82ca9d" />
      </BarChart>
    </div>
  );
}
```

---

## ⚠️ 常见问题与解决方案

### 问题 1：图表不显示

**症状**：图表区域空白或尺寸为 0。

**排查清单**：

```tsx
// ✅ 检查 1：是否设置了 width 和 height
<LineChart width={600} height={300} data={data}>

// ✅ 检查 2：使用 ResponsiveContainer 时，父元素是否有尺寸
<div style={{ width: '100%', height: 400 }}>
  <ResponsiveContainer>
    <LineChart data={data}>...

// ✅ 检查 3：data 是否为空数组或 undefined
console.log(data); // 确认有数据

// ✅ 检查 4：dataKey 是否拼写正确
<Line dataKey="value" />  // 确认 data 中有 "value" 字段
```

### 问题 2：Tooltip 不出现

```tsx
// ✅ 确保 Tooltip 组件在图表内部
<LineChart data={data}>
  <Tooltip />          {/* 必须在图表容器内 */}
  <Line dataKey="value" />
</LineChart>

// ✅ 确保有数据系列（Line/Bar/Area 等）
// Tooltip 需要数据系列来生成内容
```

### 问题 3：坐标轴标签被截断

```tsx
// 使用 margin 为坐标轴标签预留空间
<LineChart
  width={600}
  height={300}
  data={data}
  margin={{ top: 5, right: 30, left: 20, bottom: 30 }}  // 增加底部和左侧边距
>
  <XAxis dataKey="longCategoryName" angle={-45} textAnchor="end" />
  <YAxis />
</LineChart>
```

### 问题 4：颜色自定义

```tsx
// 方法一：直接设置颜色
<Line stroke="#ff0000" />
<Bar fill="#00ff00" />

// 方法二：使用 Cell 为每个数据点设置颜色
<Bar dataKey="value">
  {data.map((entry, index) => (
    <Cell key={index} fill={entry.value > 0 ? '#00C49F' : '#FF8042'} />
  ))}
</Bar>
```

---

## 📏 TypeScript 最佳实践

### 1. 使用工厂函数获得类型安全

```tsx
import { createHorizontalChart } from 'recharts';

interface SalesData {
  month: string;
  revenue: number;
  cost: number;
  profit: number;
}

// 创建类型安全的图表组件
const Charts = createHorizontalChart<SalesData, 'month', 'revenue' | 'cost' | 'profit'>();

// TypeScript 会检查 dataKey 是否正确
<Charts.LineChart data={salesData}>
  <Charts.XAxis dataKey="month" />       {/* ✅ */}
  <Charts.Line dataKey="revenue" />      {/* ✅ */}
  <Charts.Line dataKey="revnue" />       {/* ❌ 编译错误 */}
</Charts.LineChart>
```

### 2. 自定义 Tooltip 的类型

```tsx
import { TooltipProps } from 'recharts';

// 为 Tooltip 内容组件定义完整类型
const CustomTooltip = ({
  active,
  payload,
  label,
}: TooltipProps<number, string>) => {
  if (!active || !payload?.length) return null;

  return (
    <div>
      <p>{label}</p>
      {payload.map((entry) => (
        <p key={entry.name}>
          {entry.name}: {entry.value}
        </p>
      ))}
    </div>
  );
};
```

### 3. 事件回调类型

```tsx
import { CategoricalChartState } from 'recharts/types/chart/types';

const handleClick = (data: CategoricalChartState) => {
  console.log(data.activePayload);
};

<LineChart onClick={handleClick} data={data}>
```

---

## 📐 设计规范建议

### 颜色方案

为数据系列选择有区分度的颜色：

```tsx
// 推荐的颜色序列
const CHART_COLORS = [
  '#8884d8',  // 紫色
  '#82ca9d',  // 绿色
  '#ffc658',  // 黄色
  '#ff7300',  // 橙色
  '#0088FE',  // 蓝色
  '#FF8042',  // 红橙
  '#00C49F',  // 青绿
  '#FFBB28',  // 金色
];
```

### 一致的 Margin

```tsx
// 项目中统一使用相同的 margin
const CHART_MARGIN = { top: 10, right: 30, left: 10, bottom: 10 };

<LineChart margin={CHART_MARGIN} data={data}>
```

### 封装通用图表组件

```tsx
// 封装一个项目通用的图表包装器
function ChartWrapper({ children, height = 300 }) {
  return (
    <div style={{ width: '100%', height }}>
      <ResponsiveContainer>
        {children}
      </ResponsiveContainer>
    </div>
  );
}

// 使用
<ChartWrapper height={400}>
  <LineChart data={data}>
    <XAxis dataKey="name" />
    <YAxis />
    <Tooltip />
    <Line dataKey="value" />
  </LineChart>
</ChartWrapper>
```

---

## 🧪 测试图表组件

### 使用 Vitest + React Testing Library

```tsx
import { render, screen } from '@testing-library/react';
import { LineChart, Line, XAxis, YAxis } from 'recharts';

const mockData = [
  { name: 'A', value: 100 },
  { name: 'B', value: 200 },
];

test('renders line chart', () => {
  render(
    <LineChart width={600} height={300} data={mockData}>
      <XAxis dataKey="name" />
      <YAxis />
      <Line dataKey="value" />
    </LineChart>
  );

  // 检查 SVG 元素是否存在
  const svg = document.querySelector('.recharts-surface');
  expect(svg).toBeInTheDocument();
});
```

### 测试 Tooltip 交互

```tsx
import { fireEvent } from '@testing-library/react';

test('shows tooltip on hover', async () => {
  const { container } = render(
    <LineChart width={600} height={300} data={mockData}>
      <Tooltip />
      <Line dataKey="value" />
    </LineChart>
  );

  // 模拟鼠标移入图表区域
  const chartArea = container.querySelector('.recharts-wrapper');
  fireEvent.mouseMove(chartArea, { clientX: 200, clientY: 150 });

  // 检查 Tooltip 是否可见
  // 注意：Tooltip 内容可能需要等待异步渲染
});
```

---

## 📋 本章小结

| 最佳实践 | 关键要点 |
|---------|---------|
| **性能** | 大数据关闭动画和 dot；使用 useMemo 缓存数据；避免内联对象 |
| **ResponsiveContainer** | 父元素必须有明确尺寸；优先使用 ResponsiveContainer |
| **TypeScript** | 使用 createHorizontalChart 获得类型安全；为 Tooltip 定义类型 |
| **多 Y 轴** | 使用 yAxisId 绑定数据系列到对应 Y 轴 |
| **自定义 Tooltip** | 通过 content prop 传入自定义 React 组件 |
| **图表同步** | 多个图表使用相同 syncId 实现联动 |
| **测试** | 使用 React Testing Library 测试渲染和交互 |
| **规范化** | 统一颜色方案、margin 配置；封装通用图表包装器 |

---

## 🎓 恭喜！

你已经完成了前端开发者路线的全部学习！现在你应该能够：

- ✅ 使用 Recharts 创建各种类型的图表
- ✅ 理解内部架构和设计模式
- ✅ 阅读和理解 Recharts 的核心源码
- ✅ 在实际项目中高效地使用 Recharts
- ✅ 排查和解决常见问题

---

## 📖 延伸阅读

→ [扩展学习资源](../further-learning.md)
