# JustCopy
import org.apache.commons.compress.archivers.tar.TarArchiveEntry;
import org.apache.commons.compress.archivers.tar.TarArchiveOutputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorOutputStream;
import org.springframework.stereotype.Service;

import java.io.*;
import java.nio.file.Path;
import java.nio.file.Paths;

@Service
public class FileToTarGzService {

    public void convertToTarGz(String inputFilePath, String outputTarGzPath) {
        try {
            File inputFile = new File(inputFilePath);
            String fileName = inputFile.getName();

            // Set up output streams
            FileOutputStream fos = new FileOutputStream(outputTarGzPath);
            GzipCompressorOutputStream gzipOS = new GzipCompressorOutputStream(fos);
            TarArchiveOutputStream tarOS = new TarArchiveOutputStream(gzipOS);

            // Create a TarEntry for the input file
            TarArchiveEntry tarEntry = new TarArchiveEntry(fileName);
            tarEntry.setSize(inputFile.length());
            tarOS.putArchiveEntry(tarEntry);

            // Read the content of the input file and write it to the archive
            FileInputStream fis = new FileInputStream(inputFile);
            byte[] buffer = new byte[1024];
            int len;
            while ((len = fis.read(buffer)) != -1) {
                tarOS.write(buffer, 0, len);
            }

            fis.close();
            tarOS.closeArchiveEntry();
            tarOS.close();
            gzipOS.close();
            fos.close();

            System.out.println("File " + inputFilePath + " has been converted to " + outputTarGzPath);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
