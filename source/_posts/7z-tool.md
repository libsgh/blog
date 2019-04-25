---
title: sevenzipjbinding解压7z文件
categories: java
date: 2018-10-12 17:47:55
tags: 
     - 7z
     - java
comments: true
---
common-compress也可以实现7z的解压，但是在我的场景下，某些文件会报错无法正常解压

<!-- more -->

### sevenzipjbind官网
[sevenzipjbind](http://sevenzipjbind.sourceforge.net)

### pom.xml中引用

```xml
<dependency>
    <groupId>net.sf.sevenzipjbinding</groupId>
    <artifactId>sevenzipjbinding</artifactId>
    <version>9.20-2.00beta</version>
</dependency>
<dependency>
    <groupId>net.sf.sevenzipjbinding</groupId>
    <artifactId>sevenzipjbinding-all-platforms</artifactId>
    <version>9.20-2.00beta</version>
</dependency>
```

### 工具类

```java
/**
 * 
 * @Description (解压7z)
 * @param file7zPath(7z文件路径)
 * @param outPutPath(解压路径)
 * @param passWord(文件密码.没有可随便写,或空)
 * @return
 * @throws Exception
 */
public static int un7z(String file7zPath, final String outPutPath, String passWord) throws Exception {
    IInArchive archive;
    RandomAccessFile randomAccessFile;
    randomAccessFile = new RandomAccessFile(file7zPath, "r");
    archive = SevenZip.openInArchive(null, new RandomAccessFileInStream(randomAccessFile), passWord);
    int numberOfItems = archive.getNumberOfItems();
    ISimpleInArchive simpleInArchive = archive.getSimpleInterface();
    for (final ISimpleInArchiveItem item : simpleInArchive.getArchiveItems()) {
        final int[] hash = new int[] { 0 };
        if (!item.isFolder()) {
            ExtractOperationResult result;
            final long[] sizeArray = new long[1];
            result = item.extractSlow(new ISequentialOutStream() {
                public int write(byte[] data) throws SevenZipException {
                    try {
                        String str = item.getPath();
                        str = str.substring(0, str.lastIndexOf(File.separator));
                        File file = new File(outPutPath + File.separator + str + File.separator);
                        if (!file.exists()) {
                            file.mkdirs();
                        }
                        File file1 = new File(outPutPath + File.separator + item.getPath());
                        OutputStream outputStream = new FileOutputStream(file1, true);
                        IOUtils.write(data, outputStream);
                        outputStream.close();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    hash[0] ^= Arrays.hashCode(data); // Consume data
                    sizeArray[0] += data.length;
                    return data.length; // Return amount of consumed
                }
            }, passWord);
            if (result == ExtractOperationResult.OK) {
                logger.debug("解压成功...." + String.format("%9X | %10s | %s", hash[0], sizeArray[0], item.getPath()));
            } else {
                logger.error("解压失败：密码错误或者其他错误...." + result);
            }
        }
    }
    archive.close();
    randomAccessFile.close();
    return numberOfItems;
}
```

**zip解压，压缩我直接用了hutool，很方便**

