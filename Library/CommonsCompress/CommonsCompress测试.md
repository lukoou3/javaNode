## CommonsCompress测试
要知道打包和压缩的区别，打包和压缩可以结合，只需包装一下即可。

### 基本的测试

#### CompressTest
```java
import org.apache.commons.compress.compressors.bzip2.BZip2CompressorInputStream;
import org.apache.commons.compress.compressors.bzip2.BZip2CompressorOutputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorInputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorOutputStream;
import org.apache.commons.compress.compressors.gzip.GzipParameters;
import org.apache.commons.compress.compressors.lz4.FramedLZ4CompressorOutputStream;
import org.apache.commons.compress.compressors.lzma.LZMACompressorOutputStream;
import org.apache.commons.compress.compressors.snappy.FramedSnappyCompressorOutputStream;
import org.apache.commons.compress.compressors.snappy.SnappyCompressorOutputStream;
import org.apache.commons.compress.compressors.xz.XZCompressorOutputStream;
import org.apache.commons.compress.compressors.zstandard.ZstdCompressorOutputStream;
import org.apache.commons.io.FilenameUtils;
import org.apache.commons.io.IOUtils;
import org.junit.Test;

import java.io.*;
import java.util.zip.Deflater;

public class CompressTest {
    private int bufferLen = 1024 * 1024 * 10;//buffer size: 10MByte

    @Test
    public void testGzipCompress() throws IOException{
        File srcFile = new File("F:\\test\\bb.tar");
        File destFile = new File("F:\\test\\bb.tar.gz");
        doGzipCompress(srcFile, destFile);
    }

    @Test
    public void testGzipDeCompress() throws IOException{
        File srcFile = new File("F:\\test\\bb.tar.gz");
        File destDir = new File("F:\\test");
        doGzipDecompress(srcFile, destDir);
    }

    @Test
    public void testBZip2Compress() throws IOException{
        File srcFile = new File("F:\\test\\bb.tar");
        File destFile = new File("F:\\test\\bb.tar.bz2");
        doBZip2Compress(srcFile, destFile);
    }

    @Test
    public void testFramedSnappyCompress() throws IOException{
        File srcFile = new File("F:\\test\\bb.tar");
        File destFile = new File("F:\\test\\bb.tar.fsz");
        doFramedSnappyCompress(srcFile, destFile);
    }

    @Test
    public void testSnappyCompress() throws IOException{
        File srcFile = new File("F:\\test\\bb.tar");
        File destFile = new File("F:\\test\\bb.tar.sz");
        doSnappyCompress(srcFile, destFile);
    }

    @Test
    public void testLZ4Compress() throws IOException{
        File srcFile = new File("F:\\test\\bb.tar");
        File destFile = new File("F:\\test\\bb.tar.lz4");
        doLZ4Compress(srcFile, destFile);
    }

    @Test
    public void testLZMACompress() throws IOException{
        File srcFile = new File("F:\\test\\bb.tar");
        File destFile = new File("F:\\test\\bb.tar.lzma");
        doLZMACompress(srcFile, destFile);
    }

    @Test
    public void testXZCompress() throws IOException{
        File srcFile = new File("F:\\test\\bb.tar");
        File destFile = new File("F:\\test\\bb.tar.xz");
        doXZCompress(srcFile, destFile);
    }

    @Test
    public void testZstdCompress() throws IOException{
        File srcFile = new File("F:\\test\\bb.tar");
        File destFile = new File("F:\\test\\bb.tar.zstd");
        doZstdCompress(srcFile, destFile);
    }


    protected void doGzipCompress(File srcFile, File destFile) throws IOException {
        OutputStream out = null;
        InputStream is = null;
        try {
            is = new BufferedInputStream(new FileInputStream(srcFile), bufferLen);
            GzipParameters gzipParameters = new GzipParameters();
            gzipParameters.setCompressionLevel(Deflater.DEFAULT_COMPRESSION);
            out = new GzipCompressorOutputStream(new BufferedOutputStream(new FileOutputStream(destFile), bufferLen),
                    gzipParameters);
            IOUtils.copy(is, out);
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(out);
        }
    }

    protected void doGzipDecompress(File srcFile, File destDir) throws IOException {
        InputStream is = null;
        OutputStream os = null;
        try {
            File destFile = new File(destDir, FilenameUtils.getBaseName(srcFile.toString()));
            is = new GzipCompressorInputStream(new BufferedInputStream(new FileInputStream(srcFile), bufferLen));
            os = new BufferedOutputStream(new FileOutputStream(destFile), bufferLen);
            IOUtils.copy(is, os);
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(os);
        }
    }



    public void doBZip2Decompress(File srcFile, File destDir) throws IOException {
        InputStream is = null;
        OutputStream os = null;
        try {
            File destFile = new File(destDir, FilenameUtils.getBaseName(srcFile.toString()));
            is = new BZip2CompressorInputStream(new BufferedInputStream(new FileInputStream(srcFile), bufferLen));
            os = new BufferedOutputStream(new FileOutputStream(destFile), bufferLen);
            IOUtils.copy(is, os);
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(os);
        }
    }

    public void doBZip2Compress(File srcFile, File destFile) throws IOException {
        OutputStream out = null;
        InputStream is = null;
        try {
            is = new BufferedInputStream(new FileInputStream(srcFile), bufferLen);
            out = new BZip2CompressorOutputStream(new BufferedOutputStream(new FileOutputStream(destFile), bufferLen));
            IOUtils.copy(is, out);
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(out);
        }
    }

    public void doSnappyCompress(File srcFile, File destFile) throws IOException {
        OutputStream out = null;
        InputStream is = null;
        try {
            is = new BufferedInputStream(new FileInputStream(srcFile), bufferLen);
            out = new SnappyCompressorOutputStream(new BufferedOutputStream(new FileOutputStream(destFile), bufferLen), 0L);
            IOUtils.copy(is, out);
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(out);
        }
    }

    public void doFramedSnappyCompress(File srcFile, File destFile) throws IOException {
        OutputStream out = null;
        InputStream is = null;
        try {
            is = new BufferedInputStream(new FileInputStream(srcFile), bufferLen);
            out = new FramedSnappyCompressorOutputStream(new BufferedOutputStream(new FileOutputStream(destFile), bufferLen));
            IOUtils.copy(is, out);
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(out);
        }
    }

    public void doLZ4Compress(File srcFile, File destFile) throws IOException {
        OutputStream out = null;
        InputStream is = null;
        try {
            is = new BufferedInputStream(new FileInputStream(srcFile), bufferLen);
            out = new FramedLZ4CompressorOutputStream(new BufferedOutputStream(new FileOutputStream(destFile), bufferLen));
            IOUtils.copy(is, out);
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(out);
        }
    }

    public void doLZMACompress(File srcFile, File destFile) throws IOException {
        OutputStream out = null;
        InputStream is = null;
        try {
            is = new BufferedInputStream(new FileInputStream(srcFile), bufferLen);
            out = new LZMACompressorOutputStream(new BufferedOutputStream(new FileOutputStream(destFile), bufferLen));
            IOUtils.copy(is, out);
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(out);
        }
    }

    public void doXZCompress(File srcFile, File destFile) throws IOException {
        OutputStream out = null;
        InputStream is = null;
        try {
            is = new BufferedInputStream(new FileInputStream(srcFile), bufferLen);
            out = new XZCompressorOutputStream(new BufferedOutputStream(new FileOutputStream(destFile), bufferLen));
            IOUtils.copy(is, out);
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(out);
        }
    }

    public void doZstdCompress(File srcFile, File destFile) throws IOException {
        OutputStream out = null;
        InputStream is = null;
        try {
            is = new BufferedInputStream(new FileInputStream(srcFile), bufferLen);
            out = new ZstdCompressorOutputStream(new BufferedOutputStream(new FileOutputStream(destFile), bufferLen));
            IOUtils.copy(is, out);
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(out);
        }
    }

    @Test
    public void testTime() throws IOException{
        long start = System.currentTimeMillis();
        testGzipCompress();
        long duration = System.currentTimeMillis() - start;
        System.out.println("Gzip: " + duration);

        start = start + duration;
        testBZip2Compress();
        duration = System.currentTimeMillis() - start;
        System.out.println("BZip2: " + duration);

        start = start + duration;
        testFramedSnappyCompress();
        duration = System.currentTimeMillis() - start;
        System.out.println("FramedSnappy: " + duration);

        start = start + duration;
        testSnappyCompress();
        duration = System.currentTimeMillis() - start;
        System.out.println("Snappy: " + duration);

        start = start + duration;
        testLZ4Compress();
        duration = System.currentTimeMillis() - start;
        System.out.println("LZ4: " + duration);

        start = start + duration;
        testLZMACompress();
        duration = System.currentTimeMillis() - start;
        System.out.println("LZMA: " + duration);

        start = start + duration;
        testXZCompress();
        duration = System.currentTimeMillis() - start;
        System.out.println("XZ: " + duration);

        start = start + duration;
        testZstdCompress();
        duration = System.currentTimeMillis() - start;
        System.out.println("Zstd: " + duration);
    }
}
```

#### CommonsZipTest
```java
import org.apache.commons.compress.archivers.ArchiveEntry;
import org.apache.commons.compress.archivers.ArchiveOutputStream;
import org.apache.commons.compress.archivers.zip.ZipArchiveEntry;
import org.apache.commons.compress.archivers.zip.ZipArchiveInputStream;
import org.apache.commons.compress.archivers.zip.ZipArchiveOutputStream;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.junit.Test;

import java.io.*;

public class CommonsZipTest {
    private int bufferLen = 1024 * 1024 * 10;//buffer size: 10MByte

    @Test
    public void decompress() {
        File srcFile = new File("F:\\test\\03_代码.zip");
        File destDir = new File("F:\\test");
        destDir.mkdirs();
        try {
            doDecompress(srcFile, destDir);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    @Test
    public void compress() {
        File srcFile = new File("F:\\test\\03_代码");
        File destFile = new File("F:\\test\\bb.zip");
        destFile.getParentFile().mkdirs();
        try {
            doCompress(srcFile, destFile);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    public void doDecompress(File srcFile, File destDir) throws IOException {
        ZipArchiveInputStream is = null;
        try {
            is = new ZipArchiveInputStream(new BufferedInputStream(new FileInputStream(srcFile), bufferLen));
            ZipArchiveEntry entry;
            // 遍历压缩文件中实体对象
            while ((entry = is.getNextZipEntry()) != null) {
                if (!is.canReadEntryData(entry)) {
                    // log something?
                    continue;
                }

                // 若文件为目录
                if (entry.isDirectory()) {
                    // 创建目录
                    File directory = new File(destDir, entry.getName());
                    directory.mkdirs();
                } else {
                    File file = new File(destDir, entry.getName());
                    // 创建所在目录文件夹
                    if (!file.getParentFile().exists())
                        file.getParentFile().mkdirs();
                    // 文件，复制流
                    OutputStream os = null;
                    try {
                        os = new BufferedOutputStream(new FileOutputStream(file), 1024 * 10);
                        IOUtils.copy(is, os);
                    } finally {
                        IOUtils.closeQuietly(os);
                    }
                }
            }
        } finally {
            IOUtils.closeQuietly(is);
        }
    }


    public void doCompress(File srcFile, File destFile) throws IOException {
        File compressFile;
        if (destFile.exists()) {
            if (destFile.isFile()) {
                return;
            }
            compressFile = new File(destFile, srcFile.getName() + ".zip");
        } else {
            destFile.getParentFile().mkdirs();
            compressFile = destFile;
        }

        ZipArchiveOutputStream os = null;
        InputStream is = null;
        try {
            os = new ZipArchiveOutputStream(new BufferedOutputStream(new FileOutputStream(compressFile), bufferLen));

            compressFile(os, srcFile, "");

            os.finish();
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(os);
        }
    }

    private void compressFile(ArchiveOutputStream os, File file, String compressPath) throws IOException {
        addArchiveEntry(os, file, compressPath);
        if (file.isDirectory()) {
            File[] childFiles = file.listFiles();
            if (childFiles != null) {
                String childCompressPath = StringUtils.isBlank(compressPath) ?file.getName() : (compressPath + File.separator + file.getName());
                for (File childFile : childFiles) {
                    compressFile(os, childFile, childCompressPath);
                }
            }
        }
    }

    private void addArchiveEntry(ArchiveOutputStream os, File file, String compressPath) throws IOException {
        String entryName = StringUtils.isBlank(compressPath) ? file.getName(): compressPath + File.separator + file.getName();
        ArchiveEntry entry = os.createArchiveEntry(file, entryName);
        os.putArchiveEntry(entry);
        if (file.isFile()) {
            InputStream is = null;
            try {
                is = new BufferedInputStream(new FileInputStream(file), 1024 * 10);
                IOUtils.copy(is, os);
            } finally {
                IOUtils.closeQuietly(is);
            }
        }
        os.closeArchiveEntry();
    }

}
```

#### CommonsJarTest
```java
import org.apache.commons.compress.archivers.ArchiveEntry;
import org.apache.commons.compress.archivers.ArchiveInputStream;
import org.apache.commons.compress.archivers.ArchiveOutputStream;
import org.apache.commons.compress.archivers.zip.ZipArchiveInputStream;
import org.apache.commons.compress.archivers.zip.ZipArchiveOutputStream;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.junit.Test;

import java.io.*;

//jar可以用zip解压缩，或者使用JarArchiveInputStream看源码知道这就是zip的子类没什么修改
public class CommonsJarTest {
    private int bufferLen = 1024 * 1024 * 10;//buffer size: 10MByte

    @Test
    public void decompress() {
        File srcFile = new File("F:\\test\\JetbrainsCrack-2.7-release-str.jar");
        File destDir = new File("F:\\test");
        destDir.mkdirs();
        try {
            doDecompress(srcFile, destDir);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    @Test
    public void compress() {
        File srcFile = new File("F:\\test\\03_代码");
        File destFile = new File("F:\\test\\bb.zip");
        destFile.getParentFile().mkdirs();
        try {
            doCompress(srcFile, destFile);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    public void doDecompress(File srcFile, File destDir) throws IOException {
        ArchiveInputStream is = null;
        try {
            is = new ZipArchiveInputStream(new BufferedInputStream(new FileInputStream(srcFile), bufferLen));
            ArchiveEntry entry;
            // 遍历压缩文件中实体对象
            while ((entry = is.getNextEntry()) != null) {
                if (!is.canReadEntryData(entry)) {
                    // log something?
                    continue;
                }

                // 若文件为目录
                if (entry.isDirectory()) {
                    // 创建目录
                    File directory = new File(destDir, entry.getName());
                    directory.mkdirs();
                } else {
                    File file = new File(destDir, entry.getName());
                    // 创建所在目录文件夹
                    if (!file.getParentFile().exists())
                        file.getParentFile().mkdirs();
                    // 文件，复制流
                    OutputStream os = null;
                    try {
                        os = new BufferedOutputStream(new FileOutputStream(file), 1024 * 10);
                        IOUtils.copy(is, os);
                    } finally {
                        IOUtils.closeQuietly(os);
                    }
                }
            }
        } finally {
            IOUtils.closeQuietly(is);
        }
    }


    public void doCompress(File srcFile, File destFile) throws IOException {
        File compressFile;
        if (destFile.exists()) {
            if (destFile.isFile()) {
                return;
            }
            compressFile = new File(destFile, srcFile.getName() + ".zip");
        } else {
            destFile.getParentFile().mkdirs();
            compressFile = destFile;
        }

        ArchiveOutputStream os = null;
        InputStream is = null;
        try {
            os = new ZipArchiveOutputStream(new BufferedOutputStream(new FileOutputStream(compressFile), bufferLen));

            compressFile(os, srcFile, "");

            os.finish();
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(os);
        }
    }

    private void compressFile(ArchiveOutputStream os, File file, String compressPath) throws IOException {
        addArchiveEntry(os, file, compressPath);
        if (file.isDirectory()) {
            File[] childFiles = file.listFiles();
            if (childFiles != null) {
                String childCompressPath = StringUtils.isBlank(compressPath) ?file.getName() : (compressPath + File.separator + file.getName());
                for (File childFile : childFiles) {
                    compressFile(os, childFile, childCompressPath);
                }
            }
        }
    }

    private void addArchiveEntry(ArchiveOutputStream os, File file, String compressPath) throws IOException {
        String entryName = StringUtils.isBlank(compressPath) ? file.getName(): compressPath + File.separator + file.getName();
        ArchiveEntry entry = os.createArchiveEntry(file, entryName);
        os.putArchiveEntry(entry);
        if (file.isFile()) {
            InputStream is = null;
            try {
                is = new BufferedInputStream(new FileInputStream(file), 1024 * 10);
                IOUtils.copy(is, os);
            } finally {
                IOUtils.closeQuietly(is);
            }
        }
        os.closeArchiveEntry();
    }

}
```

#### CommonsTarTest
```java
import org.apache.commons.compress.archivers.ArchiveEntry;
import org.apache.commons.compress.archivers.ArchiveInputStream;
import org.apache.commons.compress.archivers.ArchiveOutputStream;
import org.apache.commons.compress.archivers.tar.TarArchiveInputStream;
import org.apache.commons.compress.archivers.tar.TarArchiveOutputStream;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.junit.Test;

import java.io.*;

public class CommonsTarTest {
    private int bufferLen = 1024 * 1024 * 10;//buffer size: 10MByte

    @Test
    public void decompress() {
        File srcFile = new File("F:\\test\\bb.tar");
        File destDir = new File("F:\\test");
        destDir.mkdirs();
        try {
            doDecompress(srcFile, destDir);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    @Test
    public void compress() {
        File srcFile = new File("F:\\test\\03_代码");
        File destFile = new File("F:\\test\\bb.tar");
        destFile.getParentFile().mkdirs();
        try {
            doCompress(srcFile, destFile);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    public void doDecompress(File srcFile, File destDir) throws IOException {
        ArchiveInputStream is = null;
        try {
            is = new TarArchiveInputStream(new BufferedInputStream(new FileInputStream(srcFile), bufferLen));
            ArchiveEntry entry;
            // 遍历压缩文件中实体对象
            while ((entry = is.getNextEntry()) != null) {
                if (!is.canReadEntryData(entry)) {
                    // log something?
                    continue;
                }

                // 若文件为目录
                if (entry.isDirectory()) {
                    // 创建目录
                    File directory = new File(destDir, entry.getName());
                    directory.mkdirs();
                } else {
                    File file = new File(destDir, entry.getName());
                    // 创建所在目录文件夹
                    if (!file.getParentFile().exists())
                        file.getParentFile().mkdirs();
                    // 文件，复制流
                    OutputStream os = null;
                    try {
                        os = new BufferedOutputStream(new FileOutputStream(file), 1024 * 10);
                        IOUtils.copy(is, os);
                    } finally {
                        IOUtils.closeQuietly(os);
                    }
                }
            }
        } finally {
            IOUtils.closeQuietly(is);
        }
    }


    public void doCompress(File srcFile, File destFile) throws IOException {
        File compressFile;
        if (destFile.exists()) {
            if (destFile.isFile()) {
                return;
            }
            compressFile = new File(destFile, srcFile.getName() + ".zip");
        } else {
            destFile.getParentFile().mkdirs();
            compressFile = destFile;
        }

        ArchiveOutputStream os = null;
        InputStream is = null;
        try {
            os = new TarArchiveOutputStream(new BufferedOutputStream(new FileOutputStream(compressFile), bufferLen));
            //不设置文件名过长会报错
            ((TarArchiveOutputStream)os).setLongFileMode(TarArchiveOutputStream.LONGFILE_GNU);

            compressFile(os, srcFile, "");

            os.finish();
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(os);
        }
    }

    private void compressFile(ArchiveOutputStream os, File file, String compressPath) throws IOException {
        addArchiveEntry(os, file, compressPath);
        if (file.isDirectory()) {
            File[] childFiles = file.listFiles();
            if (childFiles != null) {
                String childCompressPath = StringUtils.isBlank(compressPath) ?file.getName() : (compressPath + File.separator + file.getName());
                for (File childFile : childFiles) {
                    compressFile(os, childFile, childCompressPath);
                }
            }
        }
    }

    private void addArchiveEntry(ArchiveOutputStream os, File file, String compressPath) throws IOException {
        String entryName = StringUtils.isBlank(compressPath) ? file.getName(): compressPath + File.separator + file.getName();
        ArchiveEntry entry = os.createArchiveEntry(file, entryName);
        os.putArchiveEntry(entry);
        if (file.isFile()) {
            InputStream is = null;
            try {
                is = new BufferedInputStream(new FileInputStream(file), 1024 * 10);
                IOUtils.copy(is, os);
            } finally {
                IOUtils.closeQuietly(is);
            }
        }
        os.closeArchiveEntry();
    }
}
```

#### CommonsTarGzTest
```java
import org.apache.commons.compress.archivers.ArchiveEntry;
import org.apache.commons.compress.archivers.ArchiveInputStream;
import org.apache.commons.compress.archivers.ArchiveOutputStream;
import org.apache.commons.compress.archivers.tar.TarArchiveInputStream;
import org.apache.commons.compress.archivers.tar.TarArchiveOutputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorInputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorOutputStream;
import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.junit.Test;

import java.io.*;

public class CommonsTarGzTest {

    private int bufferLen = 1024 * 1024 * 10;//buffer size: 10MByte

    @Test
    public void compress() {
        File srcFile = new File("F:\\test\\03_代码");
        File destFile = new File("F:\\test\\03_代码.tar.gz");
        destFile.getParentFile().mkdirs();
        try {
            doCompress(srcFile, destFile);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    @Test
    public void decompress() {
        File srcFile = new File("F:\\test\\03_代码.tar.gz");
        File destDir = new File("F:\\test");
        destDir.mkdirs();
        try {
            doDecompress(srcFile, destDir);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    @Test
    public void compress1() {
        File srcFile = new File("D:\\BaiduNetdiskDownload\\尚硅谷大数据项目之电商数仓\\4.视频");
        File destFile = new File("D:\\BaiduNetdiskDownload\\尚硅谷大数据项目之电商数仓\\4.视频.tar.gz");
        destFile.getParentFile().mkdirs();
        try {
            doCompress(srcFile, destFile);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    public void doDecompress(File srcFile, File destDir) throws IOException {
        ArchiveInputStream is = null;
        try {
            InputStream gzi = new GzipCompressorInputStream(new BufferedInputStream(new FileInputStream(srcFile), bufferLen));
            is = new TarArchiveInputStream(gzi);
            ArchiveEntry entry;
            // 遍历压缩文件中实体对象
            while ((entry = is.getNextEntry()) != null) {
                if (!is.canReadEntryData(entry)) {
                    // log something?
                    continue;
                }

                // 若文件为目录
                if (entry.isDirectory()) {
                    // 创建目录
                    File directory = new File(destDir, entry.getName());
                    directory.mkdirs();
                } else {
                    File file = new File(destDir, entry.getName());
                    // 创建所在目录文件夹
                    if (!file.getParentFile().exists())
                        file.getParentFile().mkdirs();
                    // 文件，复制流
                    OutputStream os = null;
                    try {
                        os = new BufferedOutputStream(new FileOutputStream(file), 1024 * 10);
                        IOUtils.copy(is, os);
                    } finally {
                        IOUtils.closeQuietly(os);
                    }
                }
            }
        } finally {
            IOUtils.closeQuietly(is);
        }
    }

    public void doCompress(File srcFile, File destFile) throws IOException {
        File compressFile;
        if (destFile.exists()) {
            if (destFile.isFile()) {
                return;
            }
            compressFile = new File(destFile, srcFile.getName() + ".zip");
        } else {
            destFile.getParentFile().mkdirs();
            compressFile = destFile;
        }

        ArchiveOutputStream os = null;
        InputStream is = null;
        try {
            OutputStream gzo = new GzipCompressorOutputStream(new BufferedOutputStream(new FileOutputStream(compressFile), bufferLen));
            os = new TarArchiveOutputStream(gzo);
            //不设置文件名过长会报错
            ((TarArchiveOutputStream)os).setLongFileMode(TarArchiveOutputStream.LONGFILE_GNU);

            compressFile(os, srcFile, "");

            os.finish();
        } finally {
            IOUtils.closeQuietly(is);
            IOUtils.closeQuietly(os);
        }
    }

    private void compressFile(ArchiveOutputStream os, File file, String compressPath) throws IOException {
        addArchiveEntry(os, file, compressPath);
        if (file.isDirectory()) {
            File[] childFiles = file.listFiles();
            if (childFiles != null) {
                String childCompressPath = StringUtils.isBlank(compressPath) ?file.getName() : (compressPath + File.separator + file.getName());
                for (File childFile : childFiles) {
                    compressFile(os, childFile, childCompressPath);
                }
            }
        }
    }

    private void addArchiveEntry(ArchiveOutputStream os, File file, String compressPath) throws IOException {
        String entryName = StringUtils.isBlank(compressPath) ? file.getName(): compressPath + File.separator + file.getName();
        ArchiveEntry entry = os.createArchiveEntry(file, entryName);
        os.putArchiveEntry(entry);
        if (file.isFile()) {
            InputStream is = null;
            try {
                is = new BufferedInputStream(new FileInputStream(file), 1024 * 10);
                IOUtils.copy(is, os);
            } finally {
                IOUtils.closeQuietly(is);
            }
        }
        os.closeArchiveEntry();
    }
}
```

```java

```