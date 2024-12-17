# telegram 方案

Telegram Web 的方案：

1. 不使用虚拟列表
2. 使用 "分片渲染" (Chunked Rendering)
3. 消息按时间分组，每组作为一个 chunk
4. 使用 requestAnimationFrame 分批渲染
5. 使用 IntersectionObserver 处理懒加载

```tsx
// Telegram 风格的实现示例
interface MessageChunk {
  date: string;
  messages: IMessageModel[];
}

const MessageList = () => {
  const [chunks, setChunks] = useState<MessageChunk[]>([]);
  const [visibleChunks, setVisibleChunks] = useState<Set<string>>(new Set());
  
  // 按日期对消息进行分组
  useEffect(() => {
    const groupedMessages = messageList.reduce((acc, msg) => {
      const date = new Date(msg.time).toDateString();
      if (!acc[date]) acc[date] = [];
      acc[date].push(msg);
      return acc;
    }, {} as Record<string, IMessageModel[]>);

    setChunks(Object.entries(groupedMessages).map(([date, messages]) => ({
      date,
      messages
    })));
  }, [messageList]);

  // 使用 IntersectionObserver 监控可见性
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          const chunkId = entry.target.getAttribute('data-chunk-id');
          if (chunkId) {
            setVisibleChunks(prev => {
              const next = new Set(prev);
              if (entry.isIntersecting) {
                next.add(chunkId);
              } else {
                next.delete(chunkId);
              }
              return next;
            });
          }
        });
      },
      { threshold: 0.1 }
    );

    // 观察所有 chunk
    document.querySelectorAll('[data-chunk-id]').forEach(el => {
      observer.observe(el);
    });

    return () => observer.disconnect();
  }, [chunks]);

  return (
    <div className={classes.messageList}>
      {chunks.map(chunk => (
        <div 
          key={chunk.date}
          data-chunk-id={chunk.date}
          className={classes.messageChunk}
        >
          <div className={classes.dateDivider}>{chunk.date}</div>
          {/* 只渲染可见的 chunk */}
          {visibleChunks.has(chunk.date) && chunk.messages.map(msg => (
            <MessageRenderer key={msg.ID} message={msg} />
          ))}
        </div>
      ))}
    </div>
  );
};
```
