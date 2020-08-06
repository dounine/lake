---
description: 递归输出某个目录下的所有日志文件，我们可以使用`commons-io`进行处理，避免重复造轮子
---

# 递归列所有文件

添加依赖包

```text
compile group: 'commons-io', name: 'commons-io', version: '2.6'
```

测试

```java
    @Test
    public void testFilters(){
        String outFilePath = "./logdir2";
        String fileFilters[] = {".log"};
        IOFileFilter[] ioFileFilters = new IOFileFilter[fileFilters.length];
        for (int i = 0; i < fileFilters.length; i++) {
            ioFileFilters[i] = FileFilterUtils.suffixFileFilter(fileFilters[i]);
        }
        File file = new File(outFilePath);

        if (file.isDirectory()) {
            IOFileFilter foldFilter = FileFilterUtils.and(
                    FileFilterUtils.directoryFileFilter(),
                    HiddenFileFilter.VISIBLE);
            IOFileFilter fileFilter = FileFilterUtils.and(ioFileFilters);
            Collection<File> logFiles = FileUtils.listFiles(file, fileFilter, foldFilter);

            for (File file1 : logFiles) {
                System.out.println(file1.getName());
            }
        }
    }
```

