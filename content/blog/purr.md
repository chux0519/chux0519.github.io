+++
title = "爬山算法拟合图片"
date = 2020-08-19T13:27:15+08:00
+++

上次参加 Rusty Days Hackathon 时，用 rust 实现了一下爬山算法。可以做到下面的效果。

<img style="display: inline;" src="https://raw.githubusercontent.com/chux0519/purr/master/purrmitive/assets/purr.0.png" width=200 alignment="left"/>
<img style="display: inline;" src="https://raw.githubusercontent.com/chux0519/purr/master/purrmitive/assets/purr.1.png" width=200 alignment="left"/>
<img style="display: inline;" src="https://raw.githubusercontent.com/chux0519/purr/master/purrmitive/assets/purr.2.png" width=200 alignment="left"/>
<img style="display: inline;" src="https://raw.githubusercontent.com/chux0519/purr/master/purrmitive/assets/purr.3.png" width=200 alignment="left"/>
<img style="display: inline;" src="https://raw.githubusercontent.com/chux0519/purr/master/purrmitive/assets/input.png" width=200 alignment="left"/>
<img style="display: inline;" src="https://raw.githubusercontent.com/chux0519/purr/master/purrmitive/assets/purr.4.png" width=200 alignment="left"/>
<img style="display: inline;" src="https://raw.githubusercontent.com/chux0519/purr/master/purrmitive/assets/purr.5.png" width=200 alignment="left"/>
<img style="display: inline;" src="https://raw.githubusercontent.com/chux0519/purr/master/purrmitive/assets/purr.7.png" width=200 alignment="left"/>
<img style="display: inline;" src="https://raw.githubusercontent.com/chux0519/purr/master/purrmitive/assets/purr.8.png" width=200 alignment="left"/>

项目地址: https://github.com/chux0519/purr

<!-- more -->

## 起源

这个项目是在 hackathon 期间完成的，主题大概是使用简单的规则，完成震惊的效果。我就想起几年前看过的另一个项目，叫做 [primitive](https://github.com/fogleman/primitive)，是 golang 的实现，核心就是爬山算法。于是我使用 rust 将它重新实现了一次，purr 就诞生了。

## 特性

purr 支持所有 golang 版本的基础图形，大多数的选项都直接支持，大部分的场景下，可以直接将 primitive 的 binary 替换为 purr。

另外，purr 做了一些优化，使得它比 golang 版本运行得更快，在部分情况下甚至达到了三倍速度(贝塞尔曲线)，详情见 [performance](https://github.com/chux0519/purr#about-performance)。即便如此，这个程序的算法核心依旧是 CPU 密集型，意味着它非常吃 CPU 性能，非常耗电。


## 优化细节

我在 reddit 上发了关于 purr 的帖子，有人好奇做到比 golang 快的优化细节，这里讲我想到的两个点。

第一是，原版本的实现里面，每个 worker 中保存了三份图片的 buffer，分别是原始图，当前图，添加正在随机图形后的图的 buffer。然后在计算当前尝试的分数时，过程如下：

```go
func (worker *Worker) Energy(shape Shape, alpha int) float64 {
	worker.Counter++
	lines := shape.Rasterize()
	// worker.Heatmap.Add(lines)
	color := computeColor(worker.Target, worker.Current, lines, alpha)
	copyLines(worker.Buffer, worker.Current, lines)
	drawLines(worker.Buffer, color, lines)
	return differencePartial(worker.Target, worker.Current, worker.Buffer, worker.Score, lines)
}
```

过程可以抽象为：

1. 获取扫描线
2. 计算填充颜色
3. 渲染临时 buffer 
4. 使用原始 buffer，当前图 buffer，以及临时 buffer 计算出得分

实际上这里的临时 buffer 只用于计算分数，于是我修改了一下 diff 函数，使得它直接根据原始 buffer，当前图 buffer 和扫描线以及颜色就直接计算得分。这样，便可以省去一次对临时 buffer 的渲染。爬山算法的每一步都有成千上万次这样的过程，这样节省下来的时间就更多了。


purr 的实现在这里：[src/core/hill_climb.rs#L48-L53](https://github.com/chux0519/purr/blob/64a00a5de39848a1269af56602587ca3e6710c7b/src/core/hill_climb.rs#L48-L53)


第二点可能的因素是，我使用的随机数生成器是 `SmallRng`，按照文档来说，它会选择平台上速度最快的实现，不考虑密码学和安全特性，这样的生成器也许也会带来比较多的速度加持。

`SmallRng` 的原话：

> The PRNG algorithm in SmallRng is chosen to be efficient on the current platform, without consideration for cryptography or security

## 总结

整个项目还剩一些小的边角没有打磨，后续会抽空继续完善，整体来讲，这次 hackathon 还是收益颇多的。
