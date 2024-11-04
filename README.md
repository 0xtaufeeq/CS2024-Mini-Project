# Build a Simple Java Chat Application with GUI
## Step-by-Step Guide to Using Sockets for Real-Time Messaging

`Server.java`

```java

import java.io.*;
import java.net.*;
import java.util.concurrent.CopyOnWriteArrayList;

public class Server { 
    private static final int PORT = 1234; 
    private static CopyOnWriteArrayList<ClientHandler> clients = new CopyOnWriteArrayList<>(); 

    public static void main(String[] args) { 
        ServerSocket serverSocket = null;
        try { 
            serverSocket = new ServerSocket(PORT); 
            System.out.println("Server is running and waiting for connections.."); 

            while (true) { 
                Socket clientSocket = serverSocket.accept(); 
                System.out.println("New client connected: " + clientSocket); 

                ClientHandler clientHandler = new ClientHandler(clientSocket); 
                clients.add(clientHandler); 
                new Thread(clientHandler).start(); 
            } 
        } catch (IOException e) { 
            e.printStackTrace(); 
        } finally {
            if (serverSocket != null && !serverSocket.isClosed()) {
                try {
                    serverSocket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    } 
 
    public static void broadcast(String message, ClientHandler sender) { 
        for (ClientHandler client : clients) { 
            if (client != sender) { 
                client.sendMessage(message); 
            } 
        } 
    } 

    private static class ClientHandler implements Runnable { 
        private Socket clientSocket; 
        private PrintWriter out; 
        private BufferedReader in; 
        private String Username; 

        public ClientHandler(Socket socket) { 
            this.clientSocket = socket; 

            try { 
                out = new PrintWriter(clientSocket.getOutputStream(), true); 
                in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream())); 
            } catch (IOException e) { 
                e.printStackTrace(); 
            } 
        } 

        @Override
        public void run() { 
            try { 
                Username = getUsername();
                System.out.println("User " + Username + " connected.");

                out.println("Welcome to the chat " + Username + "!");
                out.println("Type Your Message"); 
                String inputLine; 

                while ((inputLine = in.readLine()) != null) { 
                    System.out.println("[" + Username + "]: " + inputLine);
                    broadcast("[" + Username + "]: " + inputLine, this);
                } 

                clients.remove(this); 

                in.close(); 
                out.close(); 
                clientSocket.close(); 
            } catch (IOException e) { 
                e.printStackTrace(); 
            } 
        } 

        private String getUsername() throws IOException { 
            out.println("Enter your username:"); 
            return in.readLine(); 
        } 

        public void sendMessage(String message) { 
            out.println(message); 
            out.println("Type Your Message"); 
        } 
    } 
}
```


`ClientGUI.java`

```java

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.*;
import java.net.Socket;

public class ClientGUI {
    private static final String SERVER_ADDRESS = "localhost";
    private static final int SERVER_PORT = 1234;
    private Socket socket;
    private PrintWriter out;
    private BufferedReader in;
    private JTextArea textArea;
    private JTextField textField;

    public ClientGUI() {
        JFrame frame = new JFrame("Java Chat Application By Taufeeq and Vasista");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 300);

        textArea = new JTextArea();
        textArea.setEditable(false);
        frame.add(new JScrollPane(textArea), BorderLayout.CENTER);

        textField = new JTextField();
        frame.add(textField, BorderLayout.SOUTH);

        JButton sendButton = new JButton("Send");
        frame.add(sendButton, BorderLayout.EAST);

        sendButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                sendMessage();
            }
        });

        textField.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                sendMessage();
            }
        });

        frame.setVisible(true);

        connectToServer();
    }

    private void connectToServer() {
        try {
            socket = new Socket(SERVER_ADDRESS, SERVER_PORT);
            System.out.println("Connected to the chat server!");

            out = new PrintWriter(socket.getOutputStream(), true);
            in = new BufferedReader(new InputStreamReader(socket.getInputStream()));

            new Thread(() -> {
                try {
                    String serverResponse;
                    while ((serverResponse = in.readLine()) != null) {
                        textArea.append(serverResponse + "\n");
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void sendMessage() {
        String userInput = textField.getText();
        if (!userInput.isEmpty()) {
            out.println(userInput);
            textField.setText("");
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(ClientGUI::new);
    }
}

```
