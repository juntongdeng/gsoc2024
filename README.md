# Google Summer of Code 2024

**Project:** [Add support for checkpoint/restore of CORK-ed UDP socket](https://summerofcode.withgoogle.com/programs/2024/projects/QcqJ2yWK)

**Organization:** [CRIU](https://criu.org)

**Mentor:** [Pavel Tikhomirov](https://github.com/Snorch)

## Task background

UDP sockets have a UDP_CORK option. When UDP_CORK is enabled, the packets sent will accumulate until UDP_CORK is disabled, at which time they will be sent as a single large packet. This is used to avoid sending packets that are too small, similar to TCP_CORK.

Currently CRIU does not support checkpoint/restore CORK-ed UDP. This is because the Linux kernel currently does not support dumping (reading back) packets from the write queue of a UDP socket. We need to extend the Linux kernel interface to implement this feature.

## Previous attempts

I am not the first person to try to solve this issue. Before me, Bui Quang Minh and Lese Doru Calin had tried to solve this issue.

Both [Bui Quang Minh](https://lore.kernel.org/all/20210811154557.6935-1-minhquangbui99@gmail.com/T/#u) and [Lese Doru Calin](https://lore.kernel.org/all/20200416132242.GA2586@white/T/#u) tried to add UDP Repair Mode (similar to TCP Repair Mode), but both were rejected for merging by the Linux kernel upstream in the end.

Linux kernel community thought that TCP Repair Mode is not a good design, it hijacks system calls and changes the behaviour of system calls, which is not an elegant solution.

UDP_CORK is not a commonly used feature. It is unacceptable to the Linux kernel community to add inelegant code to implement UDP Repair Mode for an uncommon feature.

## My thoughts

I learned from the experience of two previous attempts. I did not continue to try to add UDP Repair mode and modify the code of UDP send/receive path because I thought it was not feasible.

Additionally, I realized that supporting checkpoint/restore kernel features through trivial and inelegant kernel modifications would be unacceptable to the Linux kernel community.

This is not a good path for CRIU to support checkpoint/restore more kernel features in the future.

Therefore, in this GSoC, I abandoned all the previous paths and methods.

My goal was to explore a completely new technical path that could support checkpoint/restore various kernel features in an elegant way. Not just for the UDP_CORK scenario.

## Checkpoint/Restore In eBPF (CRIB)

[eBPF](https://ebpf.io/) is currently widely used in networking, monitoring, security, and tracing. I think eBPF is also very suitable for checkpoint/restore scenarios. This is because bpf programs run in kernel space and can naturally read in-kernel information.

The bpf program is changeable and we no longer need trivial kernel extensions. bpf programs can use ring buffer to transfer data between kernel space and user space. We can achieve higher performance and avoid the context switching and memory copying of traditional interfaces.

The current bpf subsystem cannot be used directly for checkpoint/restore and needs to be extended.

I sent out the proof-of-concept [patch series](https://lore.kernel.org/bpf/AM6PR03MB58488045E4D0FA6AEDC8BDE099A52@AM6PR03MB5848.eurprd03.prod.outlook.com/T/#t). This patch series provides a comprehensive introduction to Checkpoint/Restore In eBPF (CRIB).

LWN.NET wrote an [article](https://lwn.net/Articles/984313/) introducing Checkpoint/Restore In eBPF (CRIB), which also includes the history of checkpoint/restore technologies.

I [talked](https://lpc.events/event/18/contributions/1812/) in the eBPF track at Linux Plumbers Conference 2024, including reviewing previous checkpoint/restore technologies, difficulties in implementing checkpoint/restore, detailed introduction to CRIB, future plans for CRIB, etc.

## Acknowledgements

Thanks to my mentor Pavel Tikhomirov, he gave me a lot of help and guidance throughout GSoC.

Thanks to CRIU community and GSoC for giving me this opportunity, it has been a great experience for me.

Thanks to Linux kernel community, thanks to Alexei Starovoitov, Kumar Kartikeya Dwivedi, Andrii Nakryiko for providing a lot of feedback on my patches.

Thanks to Linux Plumbers Conference organizers and committee, it was an amazing conference.