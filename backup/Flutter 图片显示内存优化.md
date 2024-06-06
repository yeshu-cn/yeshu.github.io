## 图片解码
图像解码是将图像文件中的数据转换为可显示的位图数据的过程。图像文件（如JPEG、PNG等）存储的是压缩或编码后的图像数据，在显示图像之前需要将这些数据解码为原始的像素数据，以便渲染引擎能够在屏幕上正确显示图像。

### 为什么需要图像解码

图像文件通常使用各种压缩算法来减小文件大小，例如JPEG使用有损压缩，PNG使用无损压缩。这些压缩数据需要解码成原始的像素数据（每个像素的颜色信息），以便在屏幕上显示。

### 解码过程中的宽高设置

在图像解码过程中设置宽高的目的是为了控制解码后的图像尺寸。通过在解码时设置目标宽高，可以减少内存使用并提高性能。以下是为什么解码时可以设置宽高的原因和好处：

1. **减少内存使用**：
   - 如果加载原始分辨率的图像，然后再缩放，会占用大量内存。而在解码时设置宽高，可以直接生成缩放后的图像，减少内存占用。

2. **提高性能**：
   - 在解码时直接缩放图像，可以减少CPU和GPU的负担，因为无需处理大尺寸的图像数据。

3. **适应屏幕分辨率**：
   - 在移动设备上，屏幕尺寸和分辨率有限，通常不需要加载超高分辨率的图像。通过在解码时设置合适的宽高，可以确保图像在显示设备上有最佳的显示效果。

## `ResizeImage`
`ResizeImage` 的核心原理是在图片解码时对图片进行缩放，而不是加载原始分辨率的图片，然后在显示时进行缩放。这可以有效减少内存使用，因为缩放后的图片数据量远小于原始图片数据量。


## `flutter_cache_manager`缓存逻辑
1. 缓存策略：支持多种缓存策略，例如时间策略（缓存多久过期），空间策略（缓存占用空间上限）等。
2. 缓存存储：缓存文件存储在本地文件系统中，通常是在应用的临时目录或持久化存储目录中。
3. 缓存管理：主要包括缓存的写入、读取、删除以及清理过期缓存等操作。

### 缓存策略和LRU的对比
`flutter_cache_manager` 的缓存清理策略确实与传统的 LRU（Least Recently Used，最少最近使用）策略非常相似，但它并不是一个严格意义上的 LRU 实现。下面解释它与标准 LRU 策略的相似之处和不同之处。

### 相似之处

1. **记录访问时间**：每次访问缓存文件时都会更新文件的最后访问时间。
2. **基于访问时间排序**：在清理缓存时，根据文件的最后访问时间排序，最久未使用的文件优先删除。
3. **按需删除**：当缓存总大小超过设定的限制时，按访问时间顺序删除文件，直到缓存总大小低于限制。

### 不同之处

1. **过期时间策略**：
   - **标准 LRU**：标准 LRU 只考虑最近访问时间，不涉及文件的过期时间。
   - **flutter_cache_manager**：除了考虑最近访问时间，还考虑文件的过期时间。文件在过期后会被删除，即使它可能是最近访问过的。

2. **空间策略**：
   - **标准 LRU**：标准 LRU 通常是一个固定大小的缓存（如一个固定大小的队列或哈希表），当缓存满时删除最久未使用的条目。
   - **flutter_cache_manager**：允许设置最大缓存存储空间，并根据需要清理超过存储限制的缓存文件。

3. **定期清理**：
   - **标准 LRU**：标准 LRU 通常不涉及定期清理的概念，而是当缓存达到限制时立即清理。
   - **flutter_cache_manager**：可能会定期进行缓存清理操作，删除过期文件和超过存储限制的文件。

### 代码示例对比

#### 标准 LRU 实现

以下是一个标准 LRU 缓存的简化实现：

```dart
class LRUCache<K, V> {
  final int capacity;
  final _cache = LinkedHashMap<K, V>();

  LRUCache(this.capacity);

  V? get(K key) {
    if (!_cache.containsKey(key)) return null;
    // 更新顺序
    final value = _cache.remove(key);
    _cache[key] = value!;
    return value;
  }

  void put(K key, V value) {
    if (_cache.containsKey(key)) {
      _cache.remove(key);
    } else if (_cache.length == capacity) {
      _cache.remove(_cache.keys.first);
    }
    _cache[key] = value;
  }
}
```

#### `flutter_cache_manager` 实现

`flutter_cache_manager` 的实现更复杂，包含文件的过期处理和定期清理策略：

```dart
class CacheStore {
  final int maxSize;
  final Duration maxAge;

  CacheStore(this.maxSize, this.maxAge);

  Future<List<CacheFile>> getAllCachedFiles() async {
    // 获取所有缓存文件的信息
  }

  Future<int> getTotalCacheSize() async {
    // 获取当前缓存总大小
  }

  Future<void> clean() async {
    var cacheFiles = await getAllCachedFiles();

    // 按最后访问时间排序
    cacheFiles.sort((a, b) => a.lastAccessed.compareTo(b.lastAccessed));

    // 删除过期文件
    for (var file in cacheFiles) {
      if (isFileExpired(file)) {
        await file.delete();
      }
    }

    // 删除超过存储限制的文件
    var totalSize = await getTotalCacheSize();
    if (totalSize > maxSize) {
      for (var file in cacheFiles) {
        if (totalSize <= maxSize) break;
        totalSize -= await file.length();
        await file.delete();
      }
    }
  }

  bool isFileExpired(CacheFile file) {
    return DateTime.now().difference(file.lastAccessed) > maxAge;
  }
}
```

### 结论

虽然 `flutter_cache_manager` 采用了类似 LRU 的原则（最少最近使用），但它在实现中增加了过期时间和定期清理策略，使其更加适用于文件缓存管理。这些额外的策略使 `flutter_cache_manager` 更加灵活和实用，尤其是在移动应用的实际使用场景中。

