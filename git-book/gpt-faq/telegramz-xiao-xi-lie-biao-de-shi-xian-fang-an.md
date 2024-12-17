# telegram Z 消息列表的实现方案

1. 消息列表的核心实现在 `src/components/middle/MessageList.tsx`:

```typescript
// 简化后的核心逻辑
const MessageList = () => {
  // 使用 IntersectionObserver 监听可视区域
  const observeIntersectionForLoading = useIntersectionObserver({
    rootRef: containerRef,
    throttleMs: 200,
  });

  // 处理新消息加载
  const handleLoadMore = useCallback(() => {
    if (isLoadingMessages || !hasMoreMessages) return;
    
    loadMoreMessages();
  }, [isLoadingMessages, hasMoreMessages]);

  return (
    <div ref={containerRef} className="messages-container">
      {messages.map((message) => (
        <Message
          key={message.id}
          message={message}
          observeIntersection={observeIntersectionForLoading}
        />
      ))}
    </div>
  );
};
```

2. 图片消息处理 (`src/components/common/MediaItem.tsx`):

```tsx
const MediaItem = ({ message }) => {
  const { width, height, url } = message.content;
  const [isLoaded, setIsLoaded] = useState(false);
  
  // 计算图片尺寸
  const dimensions = useMemo(() => {
    return calculateDimensions({
      width,
      height,
      containerWidth: MAX_WIDTH,
      containerHeight: MAX_HEIGHT,
    });
  }, [width, height]);

  return (
    <div 
      className="media-wrapper"
      style={{
        width: dimensions.width,
        height: dimensions.height,
      }}
    >
      {/* 占位图 */}
      {!isLoaded && (
        <div 
          className="media-placeholder"
          style={{
            paddingBottom: `${(height / width) * 100}%`
          }}
        />
      )}
      
      {/* 真实图片 */}
      <img
        src={url}
        className={`media-content ${isLoaded ? 'show' : ''}`}
        onLoad={() => setIsLoaded(true)}
        loading="lazy"
      />
    </div>
  );
};
```

3. 关键优化策略：

```typescript
// 1. 使用 CSS 包含块优化渲染
.messages-container {
  contain: content;
}

.message-item {
  contain: layout;
}

// 2. 图片预加载
const preloadImage = (url: string) => {
  if (preloadCache.has(url)) return;
  
  const img = new Image();
  img.src = url;
  preloadCache.add(url);
};

// 3. 消息分组优化
const MessageGroup = memo(({ messages }) => {
  // 同一时间段的消息打包在一起渲染
  return messages.map(msg => (
    <Message key={msg.id} message={msg} />
  ));
});
```

核心要点：

* 列表渲染：
  * 不使用虚拟列表，而是依赖浏览器自身的 DOM 回收机制
  * 使用 IntersectionObserver 处理无限加载
  * CSS contain 优化渲染性能
* 图片处理：
  * 使用占位符维持布局稳定性
  * 懒加载 + 预加载组合
  * 计算并缓存图片尺寸
* 性能优化：
  * 消息分组减少重渲染
  * 使用 CSS transform 代替定位
  * 批量更新状态
  * 合理的防抖和节流

这种实现方式比 react-window 更灵活，且能更好地处理动态内容。参考源码：https://github.com/Ajaxy/telegram-tt/blob/master/src/components/middle/MessageList.tsxhttps://github.com/Ajaxy/telegram-tt/blob/master/src/components/common/MediaItem.tsx
