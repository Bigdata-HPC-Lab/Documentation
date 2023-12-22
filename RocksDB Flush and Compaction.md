# RocksDB write path

Writepath

```bash
#compile as release mode
make release -j 32 

#db_bench_tool.cc
=>DoWrite(thread, write_mode) #write_mode can be seq or rand
	db->write
#db/db_impl_write.cc
	=>DBImpl::Write
		return WriteImpl
		=>DBImpl::WriteImpl
			=>PipelinedWriteImpl
				=>write_thread_.JoinBatchGroup
				#db/write_thread.cc
				write_thread_.JoinBatchGroup
			=>PreprocessWrite
				=> write_controller_.IsStopped() ??? IF
					=>DelayWrite
		  =>DBImpl::ConcurrentWriteToWAL
			=>WriteBatchInternal::InsertInto

Insert 속도가 compaction이랑 flush 대비 너무 빠르면 DB에서 delay를 걸어버림
DelayWrite (https://github.com/facebook/rocksdb/wiki/Write-Stalls)
space를 너무 써버리게 되니깐
DelayWrite 없에면 flush 가 늘어남
```

Compaction

```bash
BackgroundCallCompaction
// signal if
      // * made_progress -- need to wakeup DelayWrite
      // * bg_{bottom,}_compaction_scheduled_ == 0 -- need to wakeup ~DBImpl
      // * HasPendingManualCompaction -- need to wakeup RunManualCompaction
      // If none of this is true, there is no need to signal since nobody is
      // waiting for it
      bg_cv_.SignalAll();
    }
여기서 DelayWrite을 깨움??

// WriteController is controlling write stalls in our write code-path. Write
// stalls happen when compaction can't keep up with write rate.
// All of the methods here (including WriteControllerToken's destructors) need
// to be called while holding DB mutex

```

Flush

```bash
There are three scenarios where memtable flush can be triggered:

1.Memtable size exceed write_buffer_size after a write.
2.Total memtable size across all column families exceed db_write_buffer_size, or write_buffer_manager signals a flush. In this scenario the largest memtable will be flushed.
3.Total WAL file size exceed max_total_wal_size. In this scenario the memtable with the oldest data will be flushed, in order to allow the WAL file with data from this memtable to be purged.
=======================
#db_impl_compaction_flush
DBImpl:BGWorkFlush
 =>backgroundFlush
  =>FlushMemTablesToOutputFiles
		=>FlushMemTableToOutputFile
			---> define flashjob and run
#db/flush_job.cc
	FlushJob::Run
	WriteLevel0Table
	BuildTable
		BlockBasedTableBuiilder::Add
```

Options

```bash
#db_bench
#enable concurrent WAL
-enable_pipelined_write=true 

#include/rocksdb/option.h
allow_concurrent_memtable_write 

max_write_buffer_number => number of memtable (no flush when number is high)
```
