# OS_HW_6

1. 
L1 cache reference

ChatGPT Prompt- What is  L1 cache reference, and has it made advancements since 2009 with a time of 0.5 ns?

Results- No advancment. Still averages 0.5 - 1.0 ns depending on computers architecture

Branch mispredict

ChatGPT Prompt - What is Branch mispredict, and what improvments have been made to the time since 2009 with a time of 5 ns

Results - Some advancment. Higher chance of success. The actual time depends on the computers CPU frequency. Average of 5 ns to and average of 2.5 ns

L2 cache reference



Results-

Mutex lock/unlock

Results-

Main memory reference

Results-

Read 1 MB sequentially from memory

Results-

Round trip within same datacenter

Results-

Disk seek

Results-

Read 1 MB sequentially from disk

Results-

Send packet CA->Netherlands->CA

Results-


2. 
Q1. Copy-On-Write (COW) is a memory optimization technique used during the fork() system call to reduce latency when creating a new process. Instead of immediately duplicating all memory pages from the parent process to the child, COW allows both processes to initially share the same physical memory pages marked as read-only. If either process attempts to write to a shared page, the operating system creates a separate copy of that page for the writing process—a mechanism known as "copy-on-write." This approach significantly improves performance because it avoids unnecessary copying of memory, especially when the child process does not modify most of the parent's memory or immediately calls exec() to load a new program. By deferring memory duplication until it’s absolutely necessary, COW makes fork() much faster and more efficient, especially for processes with large address spaces.

Q2. Process* fork_process(Process* parent) {
    Process* child = create_new_process();

    for each page in parent->address_space {
        // Mark both parent and child page tables as read-only and COW
        if (page.is_writable) {
            set_page_flags(parent->page_table, page.vaddr, READ_ONLY | COW);
            set_page_flags(child->page_table, page.vaddr, READ_ONLY | COW);
        }

        // Share the same physical page
        child->page_table[page.vaddr] = parent->page_table[page.vaddr];

        // Increase reference count on the shared physical page
        increment_refcount(page.physical_page);
    }

    return child;
}

void page_fault_handler(Address addr, Process* proc) {
    PageTableEntry* entry = proc->page_table[addr];

    if (entry.flags contains COW) {
        PhysicalPage* old_page = entry.physical_page;

        if (get_refcount(old_page) > 1) {
            // Shared page: make a private copy
            PhysicalPage* new_page = allocate_new_page();
            copy_page_contents(old_page, new_page);
            proc->page_table[addr] = new_page;
            set_page_flags(proc->page_table, addr, WRITABLE);
            decrement_refcount(old_page);
        } else {
            // Only one reference: just make it writable
            set_page_flags(proc->page_table, addr, WRITABLE);
        }
    } else {
        // Handle non-COW faults (not shown here)
    }
}

void free_process_memory(Process* proc) {
    for each page in proc->page_table {
        PhysicalPage* phys = page.physical_page;
        decrement_refcount(phys);

        if (get_refcount(phys) == 0) {
            free_physical_page(phys);
        }

        unmap_page(proc->page_table, page.vaddr);
    }
}

