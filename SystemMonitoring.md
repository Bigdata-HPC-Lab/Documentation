# Monitoring tool

What are the style guidelines and best practices for our engineering team.

# dmesg

---

Lists all the kernel messages

```bash
#keep updating kernel message
dmesg -w
#Before 14.10
tail -f -n 30 /var/log/syslog
```

# dstat

---

A general tool to check bandwidth

```bash
dstat

#select target device
dstat -D sda1
```

# htop

---

List cpu&mem usage and more

```bash
htop
```

# Meminfo

---

List cpu&mem usage and more

```bash
watch cat /proc/meminfo
```

# Ftrace

---

Kernel function trace

```bash
#/sys/kernel/debug/trace
echo kfree > set_ftrace_filter
cat set_ftrace_filter
   kfree
echo function > current_tracer
echo 1 > options/func_stack_trace
cat trace | tail -8
```

# GDB

---

user program debugger

```bash
gdb path to binary
gdb) attach PID
gdb) break functionName
gdb) info break [//list](//list) breaks
gdb) clear functionName [//](//clear)remove function break
```

# Blktrace

Trace lba information of a specific block device (e.g., /dev/sda, /dev/nvme0n1)

```bash
blktrace -d <dev> -o - | blkparse -i - -o <trace_file>
#blktrace -d /dev/sdc -o - | blkparse -i - -o trace.txt

\

output file analysis
D - actually issued request

```

```swift
echo "Bulk load database into L0...."
bpl=10485760;mcz=2;del=300000000;levels=2;ctrig=10000000; delay=10000000; stop=10000000; wbn=30; mbc=20; \
mb=1073741824;wbs=268435456; sync=0; r=1000000000; t=1; vs=800; bs=65536; cs=1048576; of=500000; si=1000000; \
./db_bench \
  --benchmarks=fillrandom --disable_seek_compaction=1 --mmap_read=0 --statistics=1 --histogram=1 \
  --num=$r --threads=$t --value_size=$vs --block_size=$bs --cache_size=$cs --bloom_bits=10 \
  --cache_numshardbits=4 --open_files=$of --verify_checksum=1 \
  --sync=$sync --disable_wal=1 --compression_type=zlib --stats_interval=$si --compression_ratio=0.5 \
  --write_buffer_size=$wbs --target_file_size_base=$mb --max_write_buffer_number=$wbn \
  --max_background_compactions=$mbc --level0_file_num_compaction_trigger=$ctrig \
  --level0_slowdown_writes_trigger=$delay --level0_stop_writes_trigger=$stop --num_levels=$levels \
  --delete_obsolete_files_period_micros=$del --min_level_to_compress=$mcz \
  --stats_per_interval=1 --max_bytes_for_level_base=$bpl --memtablerep=vector --use_existing_db=0 \
  --disable_auto_compactions=1 --allow_concurrent_memtable_write=false
```
