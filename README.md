pipeline {
    agent any

    environment {
        PYTHON_VERSION = '3.x.x'  // specify your Python version
        POETRY_VERSION = '1.x.x'  // specify your Poetry version
        NEXUS_REPO_URL = 'https://your-nexus-repo-url/repository/pypi-repo/'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set up Python') {
            steps {
                script {
                    // Install Python
                    sh 'pip install --upgrade pip'
                    sh 'pip install virtualenv'
                    sh 'virtualenv venv'
                    sh 'source venv/bin/activate'
                }
            }
        }

        stage('Install Poetry') {
            steps {
                script {
                    // Install Poetry
                    sh 'pip install --upgrade poetry==${env.POETRY_VERSION}'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh 'poetry install'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'poetry build'
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    sh "poetry publish --repository ${env.NEXUS_REPO_URL} --build"
                }
            }
        }
    }
}











\\\\\\\\\\\\\\\\
pipeline {
    agent any

    environment {
        PYTHON_VERSION = '3.x.x'  // specify your Python version
        POETRY_VERSION = '1.x.x'  // specify your Poetry version
        NEXUS_REPO_URL = 'https://your-nexus-repo-url/repository/pypi-repo/'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Manually clone the repository
                    sh 'git clone https://your-repo-url.git .'
                }
            }
        }

        stage('Set up Python') {
            steps {
                script {
                    // Install Python
                    sh 'curl -O https://www.python.org/ftp/python/${env.PYTHON_VERSION}/python-${env.PYTHON_VERSION}-amd64.exe'
                    sh 'python-${env.PYTHON_VERSION}-amd64.exe /quiet'
                }
            }
        }

        stage('Install Poetry') {
            steps {
                script {
                    // Install Poetry
                    sh 'curl -sSL https://install.python-poetry.org | python -'
                    sh 'source $HOME/.poetry/env'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh 'poetry install'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'poetry build'
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    sh "poetry publish --repository ${env.NEXUS_REPO_URL} --build"
                }
            }
        }
    }
}








-------------------------------
import os
import requests
import csv
from datetime import datetime, timedelta

def get_access_token():
    access_token = os.getenv('GITHUB_ACCESS_TOKEN')
    if not access_token:
        raise ValueError("GitHub access token is not set. Set the 'GITHUB_ACCESS_TOKEN' environment variable.")
    return access_token

def get_last_commit_info(organization, repository, branch, access_token):
    commit_url = f'https://api.github.com/repos/{organization}/{repository}/commits?sha={branch}&per_page=1'
    headers = {'Authorization': f'token {access_token}'}
    commit_response = requests.get(commit_url, headers=headers)

    if commit_response.status_code == 200:
        commit_data = commit_response.json()[0]
        commit_date = commit_data['commit']['author']['date']
        commit_author = commit_data['commit']['author']['name']
        return commit_date, commit_author
    else:
        return None, None

def list_branches_with_last_commit_info(organization):
    access_token = get_access_token()

    # Get a list of repositories for the organization
    repo_url = f'https://api.github.com/orgs/{organization}/repos'
    headers = {'Authorization': f'token {access_token}'}
    response = requests.get(repo_url, headers=headers)

    if response.status_code == 200:
        repositories = response.json()

        # Open a CSV file for writing
        csv_filename = 'github_repositories_info.csv'
        with open(csv_filename, 'w', newline='') as csvfile:
            fieldnames = ['Repository', 'Branch', 'Last Commit Date', 'Last Commit Author']
            csv_writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

            # Write the header to the CSV file
            csv_writer.writeheader()

            # Iterate through repositories
            for repo in repositories:
                repo_name = repo['name']

                # Get a list of branches for the repository
                branch_url = f'https://api.github.com/repos/{organization}/{repo_name}/branches'
                branch_response = requests.get(branch_url, headers=headers)

                if branch_response.status_code == 200:
                    branches = branch_response.json()

                    # Iterate through branches and retrieve last commit info
                    for branch in branches:
                        branch_name = branch['name']
                        commit_date, commit_author = get_last_commit_info(organization, repo_name, branch_name, access_token)

                        # Write data to the CSV file
                        csv_writer.writerow({
                            'Repository': repo_name,
                            'Branch': branch_name,
                            'Last Commit Date': commit_date,
                            'Last Commit Author': commit_author
                        })

        print(f'Data saved to {csv_filename}')
    else:
        print(f'Error: {response.status_code} - {response.text}')

if __name__ == "__main__":
    # Replace 'your_organization' with the actual organization name
    organization_name = 'your_organization'

    list_branches_with_last_commit_info(organization_name)




-------------------------------------------------------------------




























import os
import requests
from datetime import datetime, timedelta
import pandas as pd

def get_access_token():
    access_token = os.getenv('GITHUB_ACCESS_TOKEN')
    if not access_token:
        raise ValueError("GitHub access token is not set. Set the 'GITHUB_ACCESS_TOKEN' environment variable.")
    return access_token

def get_last_commit_info(organization, repository, branch):
    access_token = get_access_token()
    
    # Get the last commit on the branch
    commit_url = f'https://api.github.com/repos/{organization}/{repository}/commits?sha={branch}&per_page=1'
    headers = {'Authorization': f'token {access_token}'}
    commit_response = requests.get(commit_url, headers=headers)

    if commit_response.status_code == 200:
        commit_data = commit_response.json()[0]
        commit_date = commit_data['commit']['author']['date']
        commit_author = commit_data['commit']['author']['name']
        return commit_date, commit_author
    else:
        return None, None

def list_branches_with_last_commit_info(organization):
    access_token = get_access_token()

    # Get a list of repositories for the organization
    repo_url = f'https://api.github.com/orgs/{organization}/repos'
    headers = {'Authorization': f'token {access_token}'}
    response = requests.get(repo_url, headers=headers)

    if response.status_code == 200:
        repositories = response.json()

        # Create a DataFrame to store the data
        data = {'Repository': [], 'Branch': [], 'Last Commit Date': [], 'Last Commit Author': []}

        # Iterate through repositories
        for repo in repositories:
            repo_name = repo['name']

            # Get a list of branches for the repository
            branch_url = f'https://api.github.com/repos/{organization}/{repo_name}/branches'
            branch_response = requests.get(branch_url, headers=headers)

            if branch_response.status_code == 200:
                branches = branch_response.json()

                # Iterate through branches and retrieve last commit info
                for branch in branches:
                    branch_name = branch['name']
                    commit_date, commit_author = get_last_commit_info(organization, repo_name, branch_name)

                    data['Repository'].append(repo_name)
                    data['Branch'].append(branch_name)
                    data['Last Commit Date'].append(commit_date)
                    data['Last Commit Author'].append(commit_author)

        # Display data in a tabular format
        df = pd.DataFrame(data)
        print(df)

        # Save data to a CSV file
        csv_filename = 'github_repositories_info.csv'
        df.to_csv(csv_filename, index=False)
        print(f'\nData saved to {csv_filename}')
    else:
        print(f'Error: {response.status_code} - {response.text}')

if __name__ == "__main__":
    organization = 'your_organization'
    
    list_branches_with_last_commit_info(organization)







logback-  in a file
--------------------------
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>yourLogFile.log</file>
        <encoder>
            <pattern>%date %level [%thread] %logger{10} [%file:%line] - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="FILE" />
    </root>
</configuration>

-----------------

logback- on console

<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>




pseudo code

Precondition: To archive(.tar.gz) all the type of files and save them
Input No. of days old, list of file locations to be archived, parent-archive folder( eg: c:/test/archive )
Ouput: 

1. get the list of locations from the application.properties which is a single comma separated value eg: c:/test/files,c:/test/test1
2. Split the location from "," and store as a list of string(each index-value is a source-location) eg: {"c://test//files","c://test//test,..."}
3. iterated over each source-location and traverse recursively to get the absolute-path of each files and store them in an arraylist eg: {"c://test//files//a.txt", "c://test//files//b.txt" ,..}
4. iterate over each file in the arrays list
5. get the lastmodified-date of each file and calculate the duration between (last-modified-date and current-date)
6. check if the duration of the file is x days or within x days(x is the no-of-days-old from application.properties)
7. if true 
	extract the filename from the absolute-path of file ('a' is extracted from c://test//files//a.txt)
	create target-folder-location for storing the archived file(eg: parent-archive-folder +"//" lastmodified-date), check if parent-archive-folder folder already exists else create them 
		eg(c:/test/archive/yyyymmdd/)
	create the archive in .tar.gz format and save the file(as extracted-filename.tar.gz) in target-folder-location
		eg(a.tar.gz) in (c:/test/archive/yyyymmdd/)

This microservice has been scheduled to run on certain defined time as mentioned in the application.properties file (using cron expression cron expression)









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






getting tar 

import org.apache.commons.compress.archivers.tar.TarArchiveEntry;
import org.apache.commons.compress.archivers.tar.TarArchiveOutputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorOutputStream;

import java.io.*;

public class FileToTarGzConverter {

    public static void convertToTarGz(String inputFilePath, String outputFilePath) {
        try {
            File inputFile = new File(inputFilePath);
            File outputFile = new File(outputFilePath);

            // Create output directories if they don't exist
            outputFile.getParentFile().mkdirs();

            // Set up output streams
            FileOutputStream fos = new FileOutputStream(outputFile);
            GzipCompressorOutputStream gzipOS = new GzipCompressorOutputStream(fos);
            TarArchiveOutputStream tarOS = new TarArchiveOutputStream(gzipOS);

            // Create a TarEntry for the input file
            TarArchiveEntry tarEntry = new TarArchiveEntry(inputFile);
            tarEntry.setName(inputFile.getName());

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

            System.out.println("File " + inputFilePath + " has been converted to " + outputFilePath);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        // Example usage:
        String inputFilePath = "path/to/your/input/file.txt";
        String outputFilePath = "path/to/your/output/archive.tar.gz";
        convertToTarGz(inputFilePath, outputFilePath);
    }
}



ladt one


import org.apache.commons.compress.archivers.tar.TarArchiveEntry;
import org.apache.commons.compress.archivers.tar.TarArchiveOutputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorOutputStream;

import java.io.*;
import java.nio.file.FileVisitOption;
import java.nio.file.FileVisitResult;
import java.nio.file.FileVisitOption;
import java.nio.file.FileVisitResult;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.attribute.BasicFileAttributes;
import java.util.EnumSet;

public class FolderToTarGzConverter {

    public static void convertFolderToTarGz(String sourceFolderPath, String targetFolderPath) {
        try {
            Path sourcePath = new File(sourceFolderPath).toPath();
            Path targetPath = new File(targetFolderPath).toPath();

            if (!Files.exists(targetPath)) {
                Files.createDirectories(targetPath);
            }

            try (OutputStream os = new FileOutputStream(targetFolderPath + File.separator + "archive.tar.gz");
                 GzipCompressorOutputStream gzipOs = new GzipCompressorOutputStream(os);
                 TarArchiveOutputStream tarOs = new TarArchiveOutputStream(gzipOs)) {

                Files.walkFileTree(sourcePath, EnumSet.noneOf(FileVisitOption.class), Integer.MAX_VALUE, new SimpleFileVisitor<Path>() {
                    @Override
                    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                        // Create a TarArchiveEntry for each file
                        TarArchiveEntry entry = new TarArchiveEntry(file.toFile());
                        entry.setName(sourcePath.relativize(file).toString());

                        // Put the TarArchiveEntry and write the file content to the TarArchiveOutputStream
                        tarOs.putArchiveEntry(entry);
                        try (InputStream is = new FileInputStream(file.toFile())) {
                            byte[] buffer = new byte[4096];
                            int bytesRead;
                            while ((bytesRead = is.read(buffer)) != -1) {
                                tarOs.write(buffer, 0, bytesRead);
                            }
                        }
                        tarOs.closeArchiveEntry();

                        return FileVisitResult.CONTINUE;
                    }
                });
            }

            System.out.println("Folder " + sourceFolderPath + " has been converted to " + targetFolderPath);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        String sourceFolderPath = "path/to/source/folder";
        String targetFolderPath = "path/to/target/archive/folder";
        convertFolderToTarGz(sourceFolderPath, targetFolderPath);
    }
}



create archive folder

import org.apache.commons.compress.archivers.tar.TarArchiveEntry;
import org.apache.commons.compress.archivers.tar.TarArchiveOutputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorOutputStream;

import java.io.*;
import java.nio.file.*;

public class FolderToTarGzConverter {

    public static void convertFolderToTarGz(String sourceFolderPath, String targetArchiveFolderPath) {
        try {
            Path sourcePath = Paths.get(sourceFolderPath);
            Path targetPath = Paths.get(targetArchiveFolderPath);
            Path archivePath = targetPath.resolve("archive");

            // Create the archive folder if it doesn't exist
            if (!Files.exists(archivePath)) {
                Files.createDirectories(archivePath);
            }

            String archiveFileName = "archive.tar.gz";
            Path archiveFilePath = archivePath.resolve(archiveFileName);

            try (OutputStream os = new FileOutputStream(archiveFilePath.toString());
                 GzipCompressorOutputStream gzipOs = new GzipCompressorOutputStream(os);
                 TarArchiveOutputStream tarOs = new TarArchiveOutputStream(gzipOs)) {

                Files.walkFileTree(sourcePath, EnumSet.noneOf(FileVisitOption.class), Integer.MAX_VALUE, new SimpleFileVisitor<Path>() {
                    @Override
                    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                        // Create a TarArchiveEntry for each file
                        TarArchiveEntry entry = new TarArchiveEntry(file.toFile(), sourcePath.relativize(file).toString());

                        // Put the TarArchiveEntry and write the file content to the TarArchiveOutputStream
                        tarOs.putArchiveEntry(entry);
                        try (InputStream is = new FileInputStream(file.toFile())) {
                            byte[] buffer = new byte[4096];
                            int bytesRead;
                            while ((bytesRead = is.read(buffer)) != -1) {
                                tarOs.write(buffer, 0, bytesRead);
                            }
                        }
                        tarOs.closeArchiveEntry();

                        return FileVisitResult.CONTINUE;
                    }
                });
            }

            System.out.println("Folder " + sourceFolderPath + " has been converted to " + archiveFilePath);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        String sourceFolderPath = "path/to/source/folder";
        String targetArchiveFolderPath = "path/to/target/archive";
        convertFolderToTarGz(sourceFolderPath, targetArchiveFolderPath);
    }
}


hello again


import org.apache.commons.compress.archivers.tar.TarArchiveEntry;
import org.apache.commons.compress.archivers.tar.TarArchiveOutputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorOutputStream;

import java.io.*;

public class FileToTarGzConverter {

    public static void convertToTarGz(String inputFilePath, String outputFilePath) {
        try {
            File inputFile = new File(inputFilePath);
            File outputFile = new File(outputFilePath);

            // Create the parent directory for the output file if it doesn't exist
            outputFile.getParentFile().mkdirs();

            // Set up output streams
            FileOutputStream fos = new FileOutputStream(outputFile);
            GzipCompressorOutputStream gzipOS = new GzipCompressorOutputStream(fos);
            TarArchiveOutputStream tarOS = new TarArchiveOutputStream(gzipOS);

            // Create a TarEntry for the input file
            TarArchiveEntry tarEntry = new TarArchiveEntry(inputFile);
            tarEntry.setName(inputFile.getName());

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

            System.out.println("File " + inputFilePath + " has been converted to " + outputFilePath);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        // Example usage:
        String inputFilePath = "path/to/your/input/file.txt";
        String outputFilePath = "path/to/your/output/archive.tar.gz";
        convertToTarGz(inputFilePath, outputFilePath);
    }
}

recursively search 


import java.io.File;

public class FileLister {

    public static void listFilesRecursively(String directoryPath) {
        File directory = new File(directoryPath);

        if (directory.exists() && directory.isDirectory()) {
            File[] files = directory.listFiles();

            if (files != null) {
                for (File file : files) {
                    if (file.isDirectory()) {
                        // If it's a directory, traverse it recursively
                        listFilesRecursively(file.getAbsolutePath());
                    } else {
                        // If it's a file, print its absolute path
                        System.out.println("File: " + file.getAbsolutePath());
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        String startingDirectory = "path/to/starting/directory";
        listFilesRecursively(startingDirectory);
    }
}


new tarzzzzzzzz

import java.io.*;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.util.EnumSet;

import org.apache.commons.compress.archivers.tar.TarArchiveEntry;
import org.apache.commons.compress.archivers.tar.TarArchiveOutputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorOutputStream;

public class TarGzFolderConverter {

    public static void convertFolderToTarGz(String sourceFolderPath, String targetFolderPath) {
        try {
            Path sourcePath = new File(sourceFolderPath).toPath();
            Path targetPath = new File(targetFolderPath).toPath();

            if (!Files.exists(targetPath)) {
                Files.createDirectories(targetPath);
            }

            Path archiveFilePath = targetPath.resolve("archive.tar.gz");

            // Delete the existing archive file if it exists
            if (Files.exists(archiveFilePath)) {
                Files.delete(archiveFilePath);
            }

            try (OutputStream os = new FileOutputStream(archiveFilePath.toFile());
                 GzipCompressorOutputStream gzipOs = new GzipCompressorOutputStream(os);
                 TarArchiveOutputStream tarOs = new TarArchiveOutputStream(gzipOs)) {

                Files.walkFileTree(sourcePath, EnumSet.noneOf(FileVisitOption.class), Integer.MAX_VALUE, new SimpleFileVisitor<Path>() {
                    @Override
                    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                        // Create a TarArchiveEntry for each file
                        TarArchiveEntry entry = new TarArchiveEntry(file.toFile());
                        entry.setName(sourcePath.relativize(file).toString());

                        // Put the TarArchiveEntry and write the file content to the TarArchiveOutputStream
                        tarOs.putArchiveEntry(entry);
                        try (InputStream is = new FileInputStream(file.toFile())) {
                            byte[] buffer = new byte[4096];
                            int bytesRead;
                            while ((bytesRead = is.read(buffer)) != -1) {
                                tarOs.write(buffer, 0, bytesRead);
                            }
                        }
                        tarOs.closeArchiveEntry();

                        return FileVisitResult.CONTINUE;
                    }
                });
            }

            System.out.println("Folder " + sourceFolderPath + " has been converted to " + archiveFilePath);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        String sourceFolderPath = "path/to/source/folder";
        String targetFolderPath = "path/to/target/archive";
        convertFolderToTarGz(sourceFolderPath, targetFolderPath);
    }
}
