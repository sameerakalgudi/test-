# client-server-program-in-java
this program can transfer a file between two computer using ethernet connection by creation of sockets.
the servermain and directoryrcr should be in the same system,whereas the clientmain and directorytxr should be in system which acts as client
//samkalgudi//

ServerMain.java
import java.awt.BorderLayout;
import java.awt.FlowLayout;
import java.awt.GraphicsConfiguration;
import java.awt.GridLayout;
import java.awt.Desktop;
import java.awt.Dimension;
import java.awt.Container;
import java.awt.Component;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.event.*;
import java.awt.image.*;

import javax.swing.*;
import javax.swing.border.*;
import javax.swing.event.*;
import javax.swing.tree.*;
import javax.swing.table.*;
import javax.swing.filechooser.FileSystemView;

import javax.imageio.ImageIO;

import java.util.Date;
import java.util.List;
import java.util.ArrayList;

import java.io.*;
import java.nio.channels.FileChannel;

import java.net.URL;

public class ServerMain {
	

    protected static final GraphicsConfiguration APP_TITLE = null;

	public ServerMain() {
    	class FileBrowser {

    	    /** Title of the application */
    	    public static final String APP_TITLE = "FileBro";
    	    /** Used to open/edit/print files. */
    	    private Desktop desktop;
    	    /** Provides nice icons and names for files. */
    	    private FileSystemView fileSystemView;

    	    /** currently selected File. */
    	    private File currentFile;

    	    /** Main GUI container */
    	    private JPanel gui;

    	    /** File-system tree. Built Lazily */
    	    private JTree tree;
    	    private DefaultTreeModel treeModel;

    	    /** Directory listing */
    	    private JTable table;
    	    private JProgressBar progressBar;
    	    /** Table model for File[]. */
    	    private FileTableModel fileTableModel;
    	    private ListSelectionListener listSelectionListener;
    	    private boolean cellSizesSet = false;
    	    private int rowIconPadding = 6;

    	    /* File controls. */
    	    private JButton openFile;
    	    private JButton printFile;
    	    private JButton editFile;

    	    /* File details. */
    	    private JLabel fileName;
    	    private JTextField path;
    	    private JLabel date;
    	    private JLabel size;
    	    private JCheckBox readable;
    	    private JCheckBox writable;
    	    private JCheckBox executable;
    	    private JRadioButton isDirectory;
    	    private JRadioButton isFile;

    	    /* GUI options/containers for new File/Directory creation.  Created lazily. */
    	    private JPanel newFilePanel;
    	    private JRadioButton newTypeFile;
    	    private JTextField name;


    }
    }

    public static void main(String[] args)
    {
    	DirectoryRcr dirRcr = new DirectoryRcr();
    	

//server main program//

directoryrcr

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.SocketException;


public class DirectoryRcr {

    String request = "May I send?";
    String respServer = "Yes,You can";
    String dirResponse = "Directory created...Please send files";
    String dirFailedResponse = "Failed";
    String fileHeaderRecvd = "File header received ...Send File";
    String fileReceived = "File Received";
    Socket socket = null;
    OutputStream ioStream = null;
    InputStream inStream = null;
    boolean isLive = false;
    int state = 0;
    final int initialState = 0;
    final int dirHeaderWait = 1;
    final int dirWaitState = 2;
    final int fileHeaderWaitState = 3;
    final int fileContentWaitState = 4;
    final int fileReceiveState = 5;
    final int fileReceivedState = 6;
    final int finalState = 7;
    byte[] readBuffer = new byte[1024 * 100000];
    long fileSize = 0;
    String dir = "";
    FileOutputStream foStream = null;
    int fileCount = 0;
    File dstFile = null;

    public DirectoryRcr() {
        acceptConnection();
    }

    private void acceptConnection() {
        try {
            ServerSocket server = new ServerSocket(3339);
            socket = server.accept();
            isLive = true;
            ioStream = socket.getOutputStream();
            inStream = socket.getInputStream();
            state = initialState;
            startReadThread();

        } catch (IOException io) {
            io.printStackTrace();
        }
    }

    private void startReadThread() {
        Thread readRunnable = new Thread() {
            public void run() {
                while (isLive) {
                    try {
                        int num = inStream.read(readBuffer);
                        if (num > 0) {
                            byte[] tempArray = new byte[num];
                            System.arraycopy(readBuffer, 0, tempArray, 0, num);
                            processBytes(tempArray);
                        }
                        sleep(8888);

                    } catch (SocketException s) {

                    } catch (IOException e) {
                        e.printStackTrace();
                    } catch (InterruptedException i) {
                        i.printStackTrace();
                    }
                }
            }
        };
        Thread readThread = new Thread(readRunnable);
        readThread.start();
    }

    private void processBytes(byte[] buff) throws InterruptedException {
        if (state == fileReceiveState || state == fileContentWaitState) {
            //write to file
            if (state == fileContentWaitState)
                state = fileReceiveState;
            fileSize = fileSize - buff.length;
            writeToFile(buff);


            if (fileSize == 0) {
                state = fileReceivedState;
                try {
                    foStream.close();
                } catch (IOException io) {
                    io.printStackTrace();
                }
                System.out.println("Received " + dstFile.getName());
                sendResponse(fileReceived);
                fileCount--;
                if (fileCount != 0) {
                    state = fileHeaderWaitState;
                } else {
                    System.out.println("Finished");
                    state = finalState;
                    sendResponse("Thanks");
                    Thread.sleep(2000);
                    System.exit(0);
                }

                System.out.println("Received");
            }
        } else {
            parseToUTF(buff);
        }

    }

    private void parseToUTF(byte[] data) {
        try {
            String parsedMessage = new String(data, "UTF-8");
            System.out.println(parsedMessage);
            setResponse(parsedMessage);
        } catch (UnsupportedEncodingException u) {
            u.printStackTrace();
        }

    }

    private void setResponse(String message) {
        if (message.trim().equalsIgnoreCase(request) && state == initialState) {
            sendResponse(respServer);
            state = dirHeaderWait;

        } else if (state == dirHeaderWait) {
            if (createDirectory(message)) {
                sendResponse(dirResponse);
                state = fileHeaderWaitState;
            } else {
                sendResponse(dirFailedResponse);
                System.out.println("Error occurred...Going to exit");
                System.exit(0);
            }


        } else if (state == fileHeaderWaitState) {
            createFile(message);
            state = fileContentWaitState;
            sendResponse(fileHeaderRecvd);

        } else if (message.trim().equalsIgnoreCase(dirFailedResponse)) {
            System.out.println("Error occurred ....");
            System.exit(0);
        }

    }

    private void sendResponse(String resp) {
        try {
            sendBytes(resp.getBytes("UTF-8"));
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }

    private boolean createDirectory(String dirName) {
        boolean status = false;
        dir = dirName.substring(dirName.indexOf("$") + 1, dirName.indexOf("#"));
        fileCount = Integer.parseInt(dirName.substring(dirName.indexOf("#") + 1, dirName.indexOf("&")));
        if (new File(dir).mkdir()) {
            status = true;
            System.out.println("Successfully created directory  " + dirName);
        } else if (new File(dir).mkdirs()) {
            status = true;
            System.out.println("Directories were created " + dirName);

        } else if (new File(dir).exists()) {
            status = true;
            System.out.println("Directory exists" + dirName);
        } else {
            System.out.println("Could not create directory " + dirName);
            status = false;
        }

        return status;
    }

    private void createFile(String fileName) {

        String file = fileName.substring(fileName.indexOf("&") + 1, fileName.indexOf("#"));
        String lengthFile = fileName.substring(fileName.indexOf("#") + 1, fileName.indexOf("*"));
        fileSize = Integer.parseInt(lengthFile);
        dstFile = new File(dir + "/" + file);
        try {
            foStream = new FileOutputStream(dstFile);
            System.out.println("Starting to receive " + dstFile.getName());
        } catch (FileNotFoundException fn) {
            fn.printStackTrace();
        }

    }

    private void writeToFile(byte[] buff) {
        try {
            foStream.write(buff);
        } catch (IOException io) {
            io.printStackTrace();
        }
    }

    private void sendBytes(byte[] dataBytes) {
        synchronized (socket) {
            if (ioStream != null) {
                try {
                    ioStream.write(dataBytes);
                } catch (IOException io) {
                    io.printStackTrace();
                }
            }
        }

    }

}

//this should be with servermain//

clientmain

import java.io.*;
import java.net.Socket;
import java.net.SocketException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class ClientMain {
    private DirectoryTxr transmitter = null;
    Socket clientSocket = null;
    private boolean connectedStatus = false;
    private String ipAddress;
    String srcPath = null;
    String dstPath = "";
    public ClientMain() {

    }

    public void setIpAddress(String ip) {
        this.ipAddress = ip;
    }

    public void setSrcPath(String path) {
        this.srcPath = path;
    }

    public void setDstPath(String path) {
        this.dstPath = path;
    }

    private void createConnection() {
        Runnable connectRunnable = new Runnable() {
            public void run() {
                while (!connectedStatus) {
                    try {
                        clientSocket = new Socket(ipAddress, 3339);
                        connectedStatus = true;
                        transmitter = new DirectoryTxr(clientSocket, srcPath, dstPath);
                    } catch (IOException io) {
                        io.printStackTrace();
                    }
                }

            }
        };
        Thread connectionThread = new Thread(connectRunnable);
        connectionThread.start();
    }

    public static void main(String[] args) {
        ClientMain main = new ClientMain();
        main.setIpAddress("127.0.0.1");
        main.setSrcPath("home/sam/1234");
        main.setDstPath("/home/sam/5678");
        main.createConnection();

    }
}
//client main program//

directorytxr

import java.io.*;
import java.net.Socket;
import java.net.SocketException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class DirectoryTxr {
    Socket clientSocket = null;
    String srcDir = null;
    String dstDir = null;
    byte[] readBuffer = new byte[1024];
    private InputStream inStream = null;
    private OutputStream outStream = null;
    int state = 0;
    final int permissionReqState = 1;
    final int initialState = 0;
    final int dirHeaderSendState = 2;
    final int fileHeaderSendState = 3;
    final int fileSendState = 4;
    final int fileFinishedState = 5;
    private boolean isLive = false;
    private int numFiles = 0;
    private int filePointer = 0;
    String request = "May I send?";
    String respServer = "Yes,You can";
    String dirResponse = "Directory created...Please send files";
    String fileHeaderRecvd = "File header received ...Send File";
    String fileReceived = "File Received";
    String dirFailedResponse = "Failed";
    File[] opFileList = null;

    public DirectoryTxr(Socket clientSocket, String srcDir, String dstDir) {

        try {
            this.clientSocket = clientSocket;
            inStream = clientSocket.getInputStream();
            outStream = clientSocket.getOutputStream();
            isLive = true;
            this.srcDir = srcDir;
            this.dstDir = dstDir;
            state = initialState;
            readResponse(); //starting read thread
            sendMessage(request);
            state = permissionReqState;
        } catch (IOException io) {
            io.printStackTrace();
        }


    }

    private void sendMessage(String message) {
        try {
            sendBytes(request.getBytes("UTF-8"));
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }


    /**
     * Thread to read response from server
     */
    private void readResponse() {
        Runnable readRunnable = new Runnable() {
            public void run() {
                while (isLive) {
                    try {
                        int num = inStream.read(readBuffer);
                        if (num > 0) {
                            byte[] tempArray = new byte[num];
                            System.arraycopy(readBuffer, 0, tempArray, 0, num);
                            processBytes(tempArray);
                        }
                    } catch (SocketException se) {
                        System.exit(0);
                    } catch (IOException io) {
                        io.printStackTrace();
                        isLive = false;
                    }
                }
            }
        };
        Thread readThread = new Thread(readRunnable);
        readThread.start();

    }

    private void sendDirectoryHeader() {
        File file = new File(srcDir);
        if (file.isDirectory()) {
            try {
                String[] childFiles = file.list();
                numFiles = childFiles.length;
                String dirHeader = "$" + dstDir + "#" + numFiles + "&";
                sendBytes(dirHeader.getBytes("UTF-8"));
            } catch (UnsupportedEncodingException en) {
                en.printStackTrace();
            }
        } else {
            System.out.println(srcDir + " is not a valid directory");
        }
    }


    private void sendFile(String dirName) {
        File file = new File(dirName);

        if (!file.isDirectory()) {
            try {
                int len = (int) file.length();
                int buffSize = len / 8;
                //to avoid the heap limitation
                RandomAccessFile raf = new RandomAccessFile(file, "rw");
                FileChannel channel = raf.getChannel();

                int numRead = 0;
                while (numRead >= 0) {
                    ByteBuffer buf = ByteBuffer.allocate(1024 * 100000);
                    numRead = channel.read(buf);
                    if (numRead > 0) {
                        byte[] array = new byte[numRead];
                        System.arraycopy(buf.array(), 0, array, 0, numRead);
                        sendBytes(array);
                    }
                }
                System.out.println("Finished");

            } catch (IOException io) {
                io.printStackTrace();
            }

        }

    }

    private void sendHeader(String fileName) {
        try {
            File file = new File(fileName);
            if (file.isDirectory())
                return;//avoiding child directories to avoid confusion
            //if want we can sent them recursively
            //with proper state transitions

            String header = "&" + fileName + "#" + file.length() + "*";
            sendHeader(header);

            sendBytes(header.getBytes("UTF-8"));
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }

    private void sendBytes(byte[] dataBytes) {
        synchronized (clientSocket) {
            if (outStream != null) {
                try {
                    outStream.write(dataBytes);
                    outStream.flush();
                } catch (IOException io) {
                    io.printStackTrace();
                }
            }
        }

    }

    private void processBytes(byte[] data) {
        try {
            String parsedMessage = new String(data, "UTF-8");
            System.out.println(parsedMessage);
            setResponse(parsedMessage);
        } catch (UnsupportedEncodingException u) {
            u.printStackTrace();
        }
    }

    private void setResponse(String message) {
        if (message.trim().equalsIgnoreCase(respServer) && state == permissionReqState) {
            state = dirHeaderSendState;
            sendDirectoryHeader();


        } else if (message.trim().equalsIgnoreCase(dirResponse) && state == dirHeaderSendState) {
            state = fileHeaderSendState;
            if (LocateDirectory()) {
                createAndSendHeader();
            } else {
                System.out.println("Vacant or invalid directory");
            }


        } else if (message.trim().equalsIgnoreCase(fileHeaderRecvd) && state == fileHeaderSendState) {
            state = fileSendState;
            sendFile(opFileList[filePointer].toString());
            state = fileFinishedState;
            filePointer++;

        } else if (message.trim().equalsIgnoreCase(fileReceived) && state == fileFinishedState) {
            if (filePointer < numFiles) {
                createAndSendHeader();
            }
            System.out.println("Successfully sent");

        } else if (message.trim().equalsIgnoreCase(dirFailedResponse)) {
            System.out.println("Going to exit....Error ");
            // System.exit(0);
        } else if (message.trim().equalsIgnoreCase("Thanks")) {
            System.out.println("All files were copied");
        }

    }

    private void closeSocket() {
        try {
            clientSocket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    private boolean LocateDirectory() {
        boolean status = false;
        File file = new File(srcDir);
        if (file.isDirectory()) {
            opFileList = file.listFiles();
            numFiles = opFileList.length;
            if (numFiles <= 0) {
                System.out.println("No files found");
            } else {
                status = true;
            }

        }
        return status;
    }

    private void createAndSendHeader() {
        File opFile = opFileList[filePointer];
        String header = "&" + opFile.getName() + "#" + opFile.length() + "*";
        try {
            state = fileHeaderSendState;
            sendBytes(header.getBytes("UTF-8"));

        } catch (UnsupportedEncodingException e) {

        }
    }

    private void sendListFiles() {
        createAndSendHeader();

    }
}

