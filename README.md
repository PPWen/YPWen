asmlinkage long sys_mysyscp(const char *src_file, const char *dst_file){
	struct file *srcfd, *dstfd;
	loff_t pos = 0;
	char buffer[4096];
	ssize_t rsize, wsize;
	mm_segment_t old_fs = get_fs();
	set_fs(get_ds());
	
	srcfd = filp_open(src_file, O_RDONLY, 0);
	if(IS_ERR(srcfd)){
		printk("open src_file failed!\n");
		return 1;
	}
	
	dstfd = filp_open(dst_file, O_WRONLY|O_CREAT, 0600)
	if(IS_ERR(dstfd)){
		printk("open dst_file failed!\n");
		return 2;
	}
	
	while(1){
		rsize = vfs_read(srcfd, buffer, 4096, &pos);
		if(rsize == 0) break;
		else if(rsize == -1){
			printk("read src_file failed!\n");
			return 3;
		}else{
			wsize = vfs_write(dst_file, buffer, 4096, &pos);
			if(wsize == -1){
				printk("write file failed!\n");
				return 4;
			}
		}
	} 
	filp_close(src_fd, NULL);
	filp_close(dst_fd, NULL);
	printk("copy finished!\n");
	set_fd(old_fs);
	return 0;
}
