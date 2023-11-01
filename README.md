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













wertyuiop[poiuiooooooooooooooooooooooooooooo





public class FileLastModifiedDate {
    public static String getLastModifiedDate(String filename) {
        Path filePath = Paths.get(filename);

        if (filePath.toFile().exists()) {
            try {
                BasicFileAttributes attributes = java.nio.file.Files.readAttributes(filePath, BasicFileAttributes.class);
                FileTime lastModifiedTime = attributes.lastModifiedTime();
                Date lastModifiedDate = new Date(lastModifiedTime.toMillis());

                // Format the date as "yyyyMMdd"
                SimpleDateFormat dateFormat = new SimpleDateFormat("yyyyMMdd");
                return dateFormat.format(lastModifiedDate);
            } catch (IOException e) {
                e.printStackTrace();
            }
        } else {
            System.out.println("File does not exist.");
        }

        return null; // Return null in case of an error or if the file doesn't exist
    }





    tarr



    import org.apache.commons.compress.archivers.tar.TarArchiveEntry;
import org.apache.commons.compress.archivers.tar.TarArchiveOutputStream;
import org.apache.commons.compress.utils.IOUtils;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class TarGzConverter {
    public static void main(String[] args) {
        String inputDirectoryPath = "path/to/input/directory";
        String outputFilePath = "path/to/output/archive.tar.gz";

        try {
            createTarGzArchive(inputDirectoryPath, outputFilePath);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void createTarGzArchive(String inputDirectoryPath, String outputFilePath) throws IOException {
        File output = new File(outputFilePath);
        try (FileOutputStream fileOut = new FileOutputStream(output);
             TarArchiveOutputStream tarOut = new TarArchiveOutputStream(new GzipCompressorOutputStream(fileOut))) {
            File inputDirectory = new File(inputDirectoryPath);
            addFilesToTarGz(tarOut, inputDirectory, "");
        }
    }

    private static void addFilesToTarGz(TarArchiveOutputStream tarOut, File file, String entryName) throws IOException {
        String entryNamePrefix = entryName.isEmpty() ? "" : entryName + File.separator;
        TarArchiveEntry tarEntry = new TarArchiveEntry(file, entryNamePrefix + file.getName());
        tarOut.putArchiveEntry(tarEntry);

        if (file.isFile()) {
            try (FileInputStream fileInput = new FileInputStream(file)) {
                IOUtils.copy(fileInput, tarOut);
                tarOut.closeArchiveEntry();
            }
        } else if (file.isDirectory()) {
            tarOut.closeArchiveEntry();
            File[] children = file.listFiles();
            if (children != null) {
                for (File child : children) {
                    addFilesToTarGz(tarOut, child, entryNamePrefix + file.getName());
                }
            }
        }
    }
}

