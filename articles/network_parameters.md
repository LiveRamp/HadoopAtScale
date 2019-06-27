# Linux kernel network parameters
 
(This is a short article, and could benefit from actual networking experts)

Don't forget to tune your underlying system network settings.  It's easy to focus on tuning Hadoop parameters and forget that Hadoop is running on a (presumably) Linux machine, and the default linux kernel parameter defaults err on the side of not crashing 128MB machines; if you are running 256GB machines with 72 cores, parameters can be set wildly more aggressive, for considerable network improvements.  

I won't give specific numbers here, because they depend on your own hardware, and the internet is full of great guides.  We have specifically tuned all of the following:

- net.core.netdev_max_backlog
- net.core.somaxconn
- net.core.wmem_max
- net.core.rmem_max
- net.ipv4.tcp_wmem
- net.ipv4.tcp_rmem

One setting I will call out specifically is net.ipv4.neigh.default.gc_thresh1/2/3: This one is critical for large host counts.  If you ever see "Neighbour table overflow" in your system logs, it means connections are being dropped and these neighbor tables need to be increased to larger than your host count.

