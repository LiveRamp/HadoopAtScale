# Deep node monitoring

The other errors outlined in this repository are a few of the more interesting problems LiveRamp has debugged, but the debugging process is presented as a fait accompli.  We have also tried to set ourselves up to solve problems faster by logging and monitoring everything which may have debugging value in the future.

One example script I’d like to share here is our Hadoop node [process_monitor](https://gist.github.com/bpodgursky/4c825bc481614a77e3328d232c40eda1) (available on GitHub).  This script runs every 5 minutes on every node in our cluster.  The goal here is to give us data that lets us look back in time, so if we want to know “what was happening on this Hadoop node at 2am?”, we can figure it out.  I’ll step through the script here:

```bash

IOTOP_PROCS="$(iotop -qqq -k -b -o -a -n 2 -d 5 --time | sort -r -n -k 2,2 -k 1,1 | awk '{if ($2 != pid) {pid=$2; print $0}}')"

echo -e "\n\n$(date)" >> /var/log/process-monitoring/disk_writes.out
echo "$IOTOP_PROCS" | sort -n -k 7 >> /var/log/process-monitoring/disk_writes.out
echo -e "\n\n$(date)" >> /var/log/process-monitoring/disk_reads.out
echo "$IOTOP_PROCS" | sort -n -k 5 >> /var/log/process-monitoring/disk_reads.out

```

- These iotop commands log the read and write bandwidth each process was doing during this time window, sorted by read and write respectively.

```bash
ps aux | awk '{if($3 != "0.0") print $0}' | sort -k3 -n >> /var/log/process-monitoring/cpu_usage.out
...
ps aux | awk '{if($4 != "0.0") print $0}' | sort -k4 -n >> /var/log/process-monitoring/memory_usage.out

```

- Sort each running process by CPU usage and memory usage respectively.

```bash
iostat -y -x 10 1 >> /var/log/process-monitoring/iostat.out
```

- iostat is a nifty tool to show disk throughput, await time, and utilization at any moment in time.  This command logs stats over a 10 second window.

```bash
sar -n DEV 5 1 >> /var/log/process-monitoring/network_usage.out
```

- Sar is a utility we use here to show network received and transmitted bytes per second on each attached interface.

```bash
ps auxf >> /var/log/process-monitoring/process_tree.out
```

- Similar info as earlier, but log the entire process tree.

```bash
df -h >> /var/log/process-monitoring/disk_usage.out
```
- df is a utility we use to show all mounted disks and log the free space on each.

```bash
ls -la /tmp >> /var/log/process-monitoring/tmp_files.out
```

- Explicitly list files in tmp (when debugging we sometimes heap dump on failure to /tmp -- if tmp space was full, this gives us a hint why)


```bash
LSOF="$(lsof | grep -v '.*.jar$')"
DIRS=$(find / -maxdepth 1  | grep -E "\/data[0-9]+")

while read -r D; do
  echo -e "\n$D\n" >> /var/log/process-monitoring/per_disk_writes.out
  echo "$LSOF" | grep -e "$D\/"  >> /var/log/process-monitoring/per_disk_writes.out
done <<< "$DIRS"

echo "$LSOF" | awk '{if($9 ~ "^/var/log") print $0}' >> /var/log/process-monitoring/per_disk_writes.out
echo -e "\n/tmp\n" >> /var/log/process-monitoring/per_disk_writes.out
```

Within each mounted volume, list the open files and the process to writing them.  This was critical to identifying our shuffle problems, because we can trace high-utilization disks back to specific tasks.

This script is not meant as the sum total of stats worth logging, but is a good starting point for being able to debug performance problems on a Hadoop worker node.

-----
Ben Podgursky ([GitHub](https://github.com/bpodgursky/))

![](https://s.gravatar.com/avatar/3a8d5632b6f9b74095e7867412f0a808?s=80&r=x)