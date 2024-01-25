# Write an Interpreter for Loongarch64

## [http://172.17.103.58/lixinyu/la_emu/](http://172.17.103.58/lixinyu/la_emu/)

## BUG1: interpreter stuck at init/main.c:620,start_kernel

### init/main.c:620

```c
	pr_info("[xxyy]%s:%d,%s\n",  __FILE__, __LINE__, __func__);
	mm_init();
	pr_info("[xxyy]%s:%d,%s\n",  __FILE__, __LINE__, __func__);
```

### 卡死时候打印PC， 查看内核反汇编和代码

```asm
877852 900000000053e5c4:   0045388e    srli.d  $r14,$r4,0xe
877853 900000000053e5c8:   0347fdcc    andi    $r12,$r14,0x1ff
877854 900000000053e5cc:   00455c8d    srli.d  $r13,$r4,0x17
877855 900000000053e5d0:   0041158c    slli.d  $r12,$r12,0x5
877856 900000000053e5d4:   6c009085    bgeu    $r4,$r5,144(0x90) # 900000000053e664 <reserve_bootmem_region+0xe8>
877857 900000000053e5d8:   6c0085c8    bgeu    $r14,$r8,132(0x84) # 900000000053e65c <reserve_bootmem_region+0xe0>
877858 900000000053e5dc:   260000ee    ldptr.d $r14,$r7,0
877859 900000000053e5e0:   002d39ad    alsl.d  $r13,$r13,$r14,0x3
877860 900000000053e5e4:   400079c0    beqz    $r14,120(0x78) # 900000000053e65c <reserve_bootmem_region+0xe0>
877861 900000000053e5e8:   260001ad    ldptr.d $r13,$r13,0
877862 900000000053e5ec:   0010b1ac    add.d   $r12,$r13,$r12
877863 900000000053e5f0:   40006da0    beqz    $r13,108(0x6c) # 900000000053e65c <reserve_bootmem_region+0xe0>
```

```c
void __meminit reserve_bootmem_region(phys_addr_t start, phys_addr_t end);
```

### reserve_bootmem_region 添加打印信息

```
printk: 1613298856171727,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 0, 97f407
printk: 1613298860280083,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97f440, 97f4e4
printk: 1613298861217359,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97f500, 97f524
printk: 1613298862152505,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97f540, 97f564
printk: 1613298863069755,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97f580, 97f587
printk: 1613298863996821,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97f5c0, 97f5c7
printk: 1613298864918587,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97f600, 97f603
printk: 1613298865851179,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97f640, 97f647
printk: 1613298866784433,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97f680, 97f7ef
printk: 1613298867642465,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97f800, 97fc07
printk: 1613298868634565,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97fc40, 97fc53
printk: 1613298869551949,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97fc80, 97fcef
printk: 1613298870434301,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 97fd00, 97fd3b
printk: 1613298871319305,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 980000, 49e7fff
printk: 1613298892978499,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 49f0000, 4a04c07
printk: 1613298894053495,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 4a04c40, 5004c3f
printk: 1613298896889905,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, fffff80, ffffff7
printk: 1613298897839675,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 15e000000, 15fffffff
printk: 1613298909859291,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 161ff0000, 161ff3fff
printk: 1613298910782351,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 17fff6440, 17fffe43f
printk: 1613298911753457,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 17fffe450, 17ffffafb
printk: 1613298912719531,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 17ffffb00, 17fffffca
printk: 1613298913680259,6[320]mm/page_alloc.c:1258,reserve_bootmem_region, 100000000000, 1fffffffffff
```

### 最后一项不正常`100000000000, 1fffffffffff`, 不像是合理的地址

**for_each_reserved_mem_region**

```c
#define for_each_reserved_mem_region(i, p_start, p_end)			\
	for (i = 0UL, __next_reserved_mem_region(&i, p_start, p_end);	\
	     i != (u64)ULLONG_MAX;					\
	     __next_reserved_mem_region(&i, p_start, p_end))
```

**__next_reserved_mem_region**

```c
/**
 * __next_reserved_mem_region - next function for for_each_reserved_region()
 * @idx: pointer to u64 loop variable
 * @out_start: ptr to phys_addr_t for start address of the region, can be %NULL
 * @out_end: ptr to phys_addr_t for end address of the region, can be %NULL
 *
 * Iterate over all reserved memory regions.
 */
void __init_memblock __next_reserved_mem_region(u64 *idx,
					   phys_addr_t *out_start,
					   phys_addr_t *out_end)
{
	struct memblock_type *type = &memblock.reserved;

	if (*idx < type->cnt) {
		struct memblock_region *r = &type->regions[*idx];
		phys_addr_t base = r->base;
		phys_addr_t size = r->size;

		if (out_start)
			*out_start = base;
		if (out_end)
			*out_end = base + size - 1;

		*idx += 1;
		return;
	}

	/* signal end of iteration */
	*idx = ULLONG_MAX;
}
```

### 添加打印确定现场

`printk: 1614964021170297,6[320]mm/memblock.c:896,__next_reserved_mem_region 22 100000000000, 1fffffffffff`

确定`memblock.reserved`中包含了一项错误数据

#### 查找`memblock.reserved`中错误数据的来源

- 搜索查找结果，怀疑`memblock_reserve`函数

```c
int __init_memblock memblock_reserve(phys_addr_t base, phys_addr_t size)
{
	phys_addr_t end = base + size - 1;

	memblock_dbg("memblock_reserve: [%pa-%pa] %pF\n",
		     &base, &end, (void *)_RET_IP_);

	return memblock_add_range(&memblock.reserved, base, size, MAX_NUMNODES, 0);
}
```

- 内核添加`memblock=mem`查看debug信息

```
printk: 1616567357467249,6memblock_reserve: [0x0000100000000000-0x00001fffffffffff] setup_arch+0x404/0x71c
```

- 确定插入了错误的一项

### 查找错误数据0x100000000000来源

- `memblock_reserve`调用处太多，条件打印返回地址

```c
	if (base == 0x100000000000) {
			pr_info("------- memblock_reserve: [%pa-%pa] %pF\n",
		     &base, &end, (void *)_RET_IP_);
	}
```

`------- memblock_reserve: [0x0000100000000000-0x00001fffffffffff] early_init_dt_reserve_memory_arch+0xc/0x1c`

- `early_init_fdt_scan_reserved_mem`

```c
void __init early_init_fdt_scan_reserved_mem(void)
{
	int n;
	u64 base, size;

	if (!initial_boot_params)
		return;

	/* Process header /memreserve/ fields */
	for (n = 0; ; n++) {
		fdt_get_mem_rsv(initial_boot_params, n, &base, &size);
		if (!size)
			break;
		early_init_dt_reserve_memory_arch(base, size, 0);
	}

	of_scan_flat_dt(__fdt_scan_reserved_mem, NULL);
	fdt_init_reserved_mem();
}
```

- `fdt_get_mem_rsv`

```c

#define CPU_TO_FDT64(x) ((EXTRACT_BYTE(x, 0) << 56) | (EXTRACT_BYTE(x, 1) << 48) | \
			 (EXTRACT_BYTE(x, 2) << 40) | (EXTRACT_BYTE(x, 3) << 32) | \
			 (EXTRACT_BYTE(x, 4) << 24) | (EXTRACT_BYTE(x, 5) << 16) | \
			 (EXTRACT_BYTE(x, 6) << 8) | EXTRACT_BYTE(x, 7))

int fdt_get_mem_rsv(const void *fdt, int n, uint64_t *address, uint64_t *size)
{
	FDT_CHECK_HEADER(fdt);
	*address = fdt64_to_cpu(fdt_mem_rsv_(fdt, n)->address);
	*size = fdt64_to_cpu(fdt_mem_rsv_(fdt, n)->size);
	return 0;
}
```

fdt_get_mem_rsv

```
9000000000521a60 <fdt_get_mem_rsv>:
9000000000521a60:   02ff4063    addi.d  $r3,$r3,-48(0xfd0)
9000000000521a64:   29c08077    st.d    $r23,$r3,32(0x20)
9000000000521a68:   29c06078    st.d    $r24,$r3,24(0x18)
9000000000521a6c:   29c04079    st.d    $r25,$r3,16(0x10)
9000000000521a70:   29c0207a    st.d    $r26,$r3,8(0x8)
9000000000521a74:   29c0a061    st.d    $r1,$r3,40(0x28)
9000000000521a78:   00150099    move    $r25,$r4
9000000000521a7c:   001500b7    move    $r23,$r5
9000000000521a80:   001500da    move    $r26,$r6
9000000000521a84:   001500f8    move    $r24,$r7
9000000000521a88:   57f93bff    bl  -1736(0xffff938) # 90000000005213c0 <fdt_check_header>
9000000000521a8c:   44004c80    bnez    $r4,76(0x4c) # 9000000000521ad8 <fdt_get_mem_rsv+0x78>
9000000000521a90:   2400132c    ldptr.w $r12,$r25,16(0x10)
9000000000521a94:   002de6e5    alsl.d  $r5,$r23,$r25,0x4
9000000000521a98:   0000318c    revb.2h $r12,$r12
9000000000521a9c:   004cc18c    rotri.w $r12,$r12,0x10
9000000000521aa0:   00df018c    bstrpick.d  $r12,$r12,0x1f,0x0
9000000000521aa4:   380c30ac    ldx.d   $r12,$r5,$r12
9000000000521aa8:   0000358c    revb.4h $r12,$r12
9000000000521aac:   0000458c    revh.d  $r12,$r12
9000000000521ab0:   2700034c    stptr.d $r12,$r26,0
9000000000521ab4:   2400132c    ldptr.w $r12,$r25,16(0x10)
9000000000521ab8:   0000318c    revb.2h $r12,$r12
9000000000521abc:   004cc18c    rotri.w $r12,$r12,0x10
9000000000521ac0:   00df018c    bstrpick.d  $r12,$r12,0x1f,0x0
9000000000521ac4:   0010b0ac    add.d   $r12,$r5,$r12
9000000000521ac8:   28c0218c    ld.d    $r12,$r12,8(0x8)
9000000000521acc:   0000358c    revb.4h $r12,$r12
9000000000521ad0:   0000458c    revh.d  $r12,$r12
9000000000521ad4:   2700030c    stptr.d $r12,$r24,0
```

```c
                if (env->pc >= 0x9000000000521a60 && env->pc <= 0x9000000000521af0 && env->gpr[12] == 0x0000100000000000) {
                    printf("%lx\n", env->pc);
                    exit(0);
                }
```

pc: 9000000000521aac - 4, revb.4h

### 修正revb.4h翻译，解决问题

## BUG2, BUG: Bad page state in process swapper  pfn:01442


### 源码添加打印，查看信息定位到`free_pages_check_bad`的`atomic_read(&page->_mapcount) != -1)`

```c
static void free_pages_check_bad(struct page *page)
{
	const char *bad_reason;
	unsigned long bad_flags;

	bad_reason = NULL;
	bad_flags = 0;

	if (unlikely(atomic_read(&page->_mapcount) != -1))
		bad_reason = "nonzero mapcount";
	if (unlikely(page->mapping != NULL))
		bad_reason = "non-NULL mapping";
	if (unlikely(page_ref_count(page) != 0))
		bad_reason = "nonzero _refcount";
	if (unlikely(page->flags & PAGE_FLAGS_CHECK_AT_FREE)) {
		bad_reason = "PAGE_FLAGS_CHECK_AT_FREE flag(s) set";
		bad_flags = PAGE_FLAGS_CHECK_AT_FREE;
	}
#ifdef CONFIG_MEMCG
	if (unlikely(page->mem_cgroup))
		bad_reason = "page still charged to cgroup";
#endif
	bad_page(page, bad_reason, bad_flags);
}
```

### 查看并watch 一下这个地址

```c
	if (page->_mapcount.counter != -1) {
		pr_info("[320]%s:%d,%s addr:%px value:%x\n",  __FILE__, __LINE__, __func__, &page->_mapcount, (int)page->_mapcount.counter);
	}
```

`mm/page_alloc.c:933,free_pages_check_bad addr:fffffefffe0510b0 value:ffffff7f`

- store指令打印地址转换情况

```c
    if (addr >= 0xfffffefffe0510a8ull && addr < 0xfffffefffe0510b4ull) {
        printf("store pc:%lx va:%lx,pa:%lx\n", env->pc, addr, ha);
    }
```

- 每条指令执行完后检查物理内存里的值的变化情况

```c
      if (val != (int)ram_ldw(ram, 0x15e0510b0)) {
         printf("changed : pc;%lx, old:%x, new:%x\n", env->pc, val, (int)ram_ldw(ram, 0x15e0510b0));
         val = (int)ram_ldw(ram, 0x15e0510b0);
      }
```

```
17   store pc:900000000053e960 va:fffffefffe0510b0,pa:15e0510b0
18   changed : pc;900000000053e964, old:0, new:ffffffff
19**   changed : pc;9000000000301174, old:ffffffff, new:ffffff7f
20   store pc:9000000000300db4 va:fffffefffe0510b0,pa:15e0510b0
21   changed : pc;9000000000300db8, old:ffffff7f, new:ffffffff
```

### 没有向该地址store， 值却发生了变化？？？

**怀疑其它虚地址指向该物理地址**

添加打印信息

```c
    if (ha == 0x15e0510b0) {
        printf("ha store pc:%lx va:%lx,pa:%lx\n", env->pc, addr, ha);
    }
```

```
store pc:900000000053e960 va:fffffefffe0510b0,pa:15e0510b0
ha store pc:900000000053e960 va:fffffefffe0510b0,pa:15e0510b0
changed : pc;900000000053e964, old:0, new:ffffffff
ha store pc:9000000000301170 va:fffffefffe0500b0,pa:15e0510b0
changed : pc;9000000000301174, old:ffffffff, new:ffffff7f
store pc:9000000000300db4 va:fffffefffe0510b0,pa:15e0510b0
ha store pc:9000000000300db4 va:fffffefffe0510b0,pa:15e0510b0
changed : pc;9000000000300db8, old:ffffff7f, new:ffffffff
```

### va:fffffefffe0500b0 指向了 pa:15e0510b0？？？

- 这是个大页，但是`pa` 和`va`的低位不同，TLB实现有问题。

- 查看手册， 如果是大页， 12bit表示G位，应该是忘记清除了，查找相关逻辑，找到问题






