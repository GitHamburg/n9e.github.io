---
weight: 100
title: "告警自愈"
---

所谓的告警自愈，主要是靠告警规则中配置webhook，即在告警和恢复的时候，自动调用指定的webhook地址，在这个webhook中写自动处理的逻辑。这个功能大部分监控系统都支持，不过对于运维人员，这个门槛可能稍高。夜莺和ibex（类似之前v3版本中的job模块）整合，可以做到告警的时候自动执行某个脚本，运维人员只需要写脚本即可，这个门槛会低一些。

#### 自愈脚本

在这个页面提前准备好一些自愈脚本，python、perl、ruby都行，只要机器上有对应的运行环境，系统会为每个自愈脚本生成一个ID，这个ID后面用于配置到告警规则那里形成联动。

#### 执行历史

保存了所有告警自愈的调用历史，可以查看执行结果，包括脚本的stdout和stderr等

#### 告警规则关联自愈脚本

告警规则中可以配置回调地址，比如`${ibex}/11`就表示这个回调是个特殊的回调，是要和告警自愈脚本打通，11表示的是自愈脚本的编号。如果我想在某个告警规则触发告警的时候，执行第22号脚本，就配置为：`${ibex}/22`，告警规则和自愈脚本都是要归属某个业务组的，规则关联的脚本只能是同业务组的。

默认情况下，脚本的执行是去告警的那个机器执行，如果想把脚本放在特定的机器执行，可以这么配置：`${ibex}/11/c3-center01.bj`，c3-center01.bj表示希望脚本放到这个机器执行，这个机器需要和告警规则隶属于相同的业务组。典型的是这个机器是这个业务组的中控机，在中控机上跑一些自愈逻辑。


#### 额外说明

对于告警回调的处理，这里额外交代一下，默认情况下，不管是告警消息还是恢复消息，都会调用webhook，有时，webhook只需要在告警的时候才做处理，在恢复的时候啥都不做，此时，webhook逻辑要去看IsRecovered这个字段，这个字段如果是1就表示是恢复消息，如果这个字段是0就表示是告警消息。如果是`${ibex}`这种回调，系统会自动判断，只在告警的时候才触发，恢复的时候不触发，这点也请注意。因为从使用角度，自愈脚本只需要在告警的时候运行，恢复的时候，脚本是无需执行的。