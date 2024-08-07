using System;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading;
using System.Windows;

namespace UDPServer
{
    public partial class MainWindow : Window
    {
        private UdpClient udpClient;
        private const int port = 12345;

        public MainWindow()
        {
            InitializeComponent();
            StartListening();
        }

        private void StartListening()
        {
            udpClient = new UdpClient(port);
            Thread listenThread = new Thread(new ThreadStart(Listen));
            listenThread.IsBackground = true;
            listenThread.Start();
        }

        private void Listen()
        {
            while (true)
            {
                IPEndPoint remoteEP = new IPEndPoint(IPAddress.Any, port);
                byte[] data = udpClient.Receive(ref remoteEP);
                string message = Encoding.UTF8.GetString(data);
                Dispatcher.Invoke(() => { chatBox.AppendText($"Received from {remoteEP}: {message}\n"); });

            
            }
        }

        private void Send(string message, string ipAddress)
        {
            try
            {
                byte[] data = Encoding.UTF8.GetBytes(message);
                udpClient.Send(data, data.Length, ipAddress, port);
                Dispatcher.Invoke(() => { chatBox.AppendText($"Sent to {ipAddress}: {message}\n"); });
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Error: {ex.Message}");
            }
        }

        private void sendButton_Click(object sender, RoutedEventArgs e)
        {
            string message = messageBox.Text;
            if (!string.IsNullOrWhiteSpace(message))
            {
                Send(message, "127.0.0.1"); // Change this IP address to the client's IP address
                messageBox.Text = "";
            }
        }
    }
}

////////////////////////////////////

<Window x:Class="UDPServer.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="UDP Server" Height="450" Width="525">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="*" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>

        <TextBox x:Name="chatBox" Grid.Row="0" Margin="10" TextWrapping="Wrap" VerticalScrollBarVisibility="Auto" IsReadOnly="True"/>

        <Grid Grid.Row="1" Margin="10" Grid.ColumnSpan="2">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="*" />
                <ColumnDefinition Width="Auto" />
            </Grid.ColumnDefinitions>
            <TextBox x:Name="messageBox" TextWrapping="Wrap" VerticalScrollBarVisibility="Auto"/>
            <Button x:Name="sendButton" Content="Send" Grid.Column="1" Margin="5" Padding="10" Click="sendButton_Click"/>
        </Grid>

        <TextBlock Grid.Row="2" Margin="10" HorizontalAlignment="Center">
            Server application for UDP messaging.
        </TextBlock>
    </Grid>
</Window>


-----------------------------------------------

using System;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Windows;

namespace UDPClient
{
    public partial class MainWindow : Window
    {
        private UdpClient udpClient;
        private const int port = 12345;

        public MainWindow()
        {
            InitializeComponent();
            udpClient = new UdpClient();
        }

        private void Send(string message, string ipAddress)
        {
            try
            {
                byte[] data = Encoding.UTF8.GetBytes(message);
                udpClient.Send(data, data.Length, ipAddress, port);
                chatBox.AppendText($"Sent to {ipAddress}: {message}\n");
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Error: {ex.Message}");
            }
        }

        private void sendButton_Click(object sender, RoutedEventArgs e)
        {
            string message = messageBox.Text;
            if (!string.IsNullOrWhiteSpace(message))
            {
                Send(message, "127.0.0.1"); // Change this IP address to the server's IP address
                messageBox.Text = "";
            }
        }
    }
}

////////////////////////////////////////////////////////////

<Window x:Class="UDPClient.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="UDP Client" Height="350" Width="525">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="*" />
            <RowDefinition Height="Auto" />
            <RowDefinition Height="Auto" />
        </Grid.RowDefinitions>

        <TextBox x:Name="chatBox" Grid.Row="0" Margin="10" TextWrapping="Wrap" VerticalScrollBarVisibility="Auto" IsReadOnly="True"/>

        <Grid Grid.Row="1" Margin="10" Grid.ColumnSpan="2">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="*" />
                <ColumnDefinition Width="Auto" />
            </Grid.ColumnDefinitions>
            <TextBox x:Name="messageBox" TextWrapping="Wrap" VerticalScrollBarVisibility="Auto"/>
            <Button x:Name="sendButton" Content="Send" Grid.Column="1" Margin="5" Padding="10" Click="sendButton_Click"/>
        </Grid>

        <TextBlock Grid.Row="2" Margin="10" HorizontalAlignment="Center">
            Use server's IP address for communication.
        </TextBlock>
    </Grid>
</Window>


--------

private void SendToClient(string message)
{
    try
    {
        byte[] data = Encoding.UTF8.GetBytes(message);
        udpClient.Send(data, data.Length, "127.0.0.1", port); // Change IP address if needed
        Dispatcher.Invoke(() => { chatBox.AppendText($"Sent to client: {message}\n"); });
    }
    catch (Exception ex)
    {
        MessageBox.Show($"Error: {ex.Message}");
    }
}
-------

private void Listen()
{
    while (true)
    {
        IPEndPoint remoteEP = new IPEndPoint(IPAddress.Any, port);
        byte[] data = udpClient.Receive(ref remoteEP);
        string message = Encoding.UTF8.GetString(data);
        Dispatcher.Invoke(() => 
        { 
            chatBox.AppendText($"Received from {remoteEP}: {message}\n"); 
            SendToClient($"Echo from server: {message}"); // Alınan mesajı client'a gönder
        });
    }
}

private void SendToClient(string message)
{
    try
    {
        byte[] data = Encoding.UTF8.GetBytes(message);
        udpClient.Send(data, data.Length, "127.0.0.1", port); // Change IP address if needed
        Dispatcher.Invoke(() => { chatBox.AppendText($"Sent to client: {message}\n"); });
    }
    catch (Exception ex)
    {
        MessageBox.Show($"Error: {ex.Message}");
    }
}

""""""""

private void Listen()
{
    while (true)
    {
        IPEndPoint remoteEP = new IPEndPoint(IPAddress.Any, port);
        byte[] data = udpClient.Receive(ref remoteEP);
        string message = Encoding.UTF8.GetString(data);
        string formattedMessage = $"Received from {remoteEP}: {message}"; // Mesajı formatla
        Dispatcher.Invoke(() => 
        { 
            chatBox.AppendText($"{formattedMessage}\n"); // Formatlı mesajı ekle
            SendToClient(formattedMessage); // Alınan formatlı mesajı client'a gönder
        });
    }
}



private void Send(string message, string ipAddress)
{
    try
    {
        byte[] data = Encoding.UTF8.GetBytes(message);
        udpClient.Send(data, data.Length, ipAddress, port);
    }
    catch (Exception ex)
    {
        MessageBox.Show($"Error: {ex.Message}");
    }
}

private void Receive()
{
    while (true)
    {
        IPEndPoint remoteEP = new IPEndPoint(IPAddress.Any, port);
        byte[] data = udpClient.Receive(ref remoteEP);
        string message = Encoding.UTF8.GetString(data);
        Dispatcher.Invoke(() => 
        { 
            chatBox.AppendText($"{message}\n"); // Alınan mesajı direkt olarak ekle
        });
    }
}

???????

using System;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Windows;

namespace UDPClient
{
    public partial class MainWindow : Window
    {
        private UdpClient udpClient;
        private const int port = 12345;

        public MainWindow()
        {
            InitializeComponent();
            udpClient = new UdpClient();
            StartReceiving();
        }

        private void StartReceiving()
        {
            Thread receiveThread = new Thread(new ThreadStart(Receive));
            receiveThread.IsBackground = true;
            receiveThread.Start();
        }

        private void Receive()
        {
            while (true)
            {
                IPEndPoint remoteEP = new IPEndPoint(IPAddress.Any, port);
                byte[] data = udpClient.Receive(ref remoteEP);
                string message = Encoding.UTF8.GetString(data);
                Dispatcher.Invoke(() => 
                { 
                    chatBox.AppendText($"Received from server: {message}\n"); // Add received message directly
                });
            }
        }

        private void Send(string message, string ipAddress)
        {
            try
            {
                byte[] data = Encoding.UTF8.GetBytes(message);
                udpClient.Send(data, data.Length, ipAddress, port);
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Error: {ex.Message}");
            }
        }

        private void sendButton_Click(object sender, RoutedEventArgs e)
        {
            string message = messageBox.Text;
            if (!string.IsNullOrWhiteSpace(message))
            {
                Send(message, "127.0.0.1"); // Send message to server
                messageBox.Text = "";
            }
        }
    }
}


......

private void StartListening()
{
    udpClient = new UdpClient(new IPEndPoint(IPAddress.Any, port));
    Thread listenThread = new Thread(new ThreadStart(Listen));
    listenThread.IsBackground = true;
    listenThread.Start();
}


&&&

<Window x:Class="UDP_Socket_Example.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="UDP Socket Example" Height="250" Width="400">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        
        <Label Grid.Row="0" Content="Select mode:"/>
        <RadioButton Grid.Row="1" Name="serverRadioButton" Content="Server"/>
        <RadioButton Grid.Row="2" Name="clientRadioButton" Content="Client"/>
        <Button Grid.Row="3" Content="Start" Click="StartButton_Click"/>
        <TextBox Grid.Row="4" Name="outputTextBox" VerticalScrollBarVisibility="Auto" IsReadOnly="True"/>
    </Grid>
</Window>



using System.Windows;
using UDP_Socket_Example.Core;

namespace UDP_Socket_Example
{
    public partial class MainWindow : Window
    {
        private const string ServerIP = "127.0.0.1";
        private const int ServerPort = 50000;
        private const int ClientPort = 60000;

        public MainWindow()
        {
            InitializeComponent();
        }

        private void StartButton_Click(object sender, RoutedEventArgs e)
        {
            if (serverRadioButton.IsChecked == true)
            {
                outputTextBox.AppendText("Starting server...\n");
                UDPSocket server = new UDPSocket(ServerIP, ServerPort);
                string receivedMessage = server.Echo();
                outputTextBox.AppendText($"Server received: {receivedMessage}\n");
            }
            else if (clientRadioButton.IsChecked == true)
            {
                outputTextBox.AppendText("Starting client...\n");
                UDPSocket client = new UDPSocket(ServerIP, ClientPort);
                client.Send("Hello World", ServerIP, ServerPort);
                string receivedMessage = client.Listen();
                outputTextBox.AppendText($"Client received: {receivedMessage}\n");
            }
            else
            {
                MessageBox.Show("Please select either server or client mode.");
            }
        }
    }
}

₺₺₺₺₺₺


using System;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Windows;

namespace UDP_Socket_Example
{
    public partial class MainWindow : Window
    {
        private Socket _socket;
        private const int BufferSize = 1024;
        public const int SIO_UDP_CONNRESET = -1744830452;
        private const int ServerPort = 50000;
        private const int ClientPort = 60000;

        public MainWindow()
        {
            InitializeComponent();
            _socket = new Socket(SocketType.Dgram, ProtocolType.Udp);
            _socket.IOControl((IOControlCode)SIO_UDP_CONNRESET, new byte[] { 0, 0, 0, 0 }, null);
        }

        private void StartButton_Click(object sender, RoutedEventArgs e)
        {
            if (serverRadioButton.IsChecked == true)
            {
                outputTextBox.AppendText("Starting server...\n");
                StartServer(ServerPort);
            }
            else if (clientRadioButton.IsChecked == true)
            {
                outputTextBox.AppendText("Starting client...\n");
                StartClient("Hello World", ServerPort);
            }
            else
            {
                MessageBox.Show("Please select either server or client mode.");
            }
        }

        private void StartServer(int port)
        {
            try
            {
                IPAddress ipAddress = IPAddress.Any;
                IPEndPoint localEndPoint = new IPEndPoint(ipAddress, port);
                _socket.Bind(localEndPoint);
                byte[] receivedBytes = new byte[BufferSize];
                EndPoint clientEndPoint = new IPEndPoint(IPAddress.Any, 0);
                int bytesRead = _socket.ReceiveFrom(receivedBytes, ref clientEndPoint);
                string receivedMessage = Encoding.ASCII.GetString(receivedBytes, 0, bytesRead);
                outputTextBox.AppendText($"Server received: {receivedMessage}\n");
                _socket.SendTo(receivedBytes, clientEndPoint);
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Error: {ex.Message}");
            }
        }

        private void StartClient(string messageToSend, int port)
        {
            try
            {
                IPAddress ipAddress = IPAddress.Parse("127.0.0.1");
                IPEndPoint serverEndPoint = new IPEndPoint(ipAddress, port);
                byte[] bytesToSend = Encoding.ASCII.GetBytes(messageToSend);
                _socket.SendTo(bytesToSend, serverEndPoint);
                byte[] receivedBytes = new byte[BufferSize];
                EndPoint serverEP = (EndPoint)serverEndPoint;
                int bytesRead = _socket.ReceiveFrom(receivedBytes, ref serverEP);
                string receivedMessage = Encoding.ASCII.GetString(receivedBytes, 0, bytesRead);
                outputTextBox.AppendText($"Client received: {receivedMessage}\n");
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Error: {ex.Message}");
            }
        }
    }
}




<Window x:Class="UDP_Socket_Example.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="UDP Socket Example" Height="300" Width="400">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <Label Grid.Row="0" Content="Select mode:"/>
        <RadioButton Grid.Row="1" Name="serverRadioButton" Content="Server"/>
        <RadioButton Grid.Row="2" Name="clientRadioButton" Content="Client"/>
        <Button Grid.Row="3" Content="Start" Click="StartButton_Click"/>
        <TextBox Grid.Row="4" Name="outputTextBox" VerticalScrollBarVisibility="Auto" IsReadOnly="True"/>
    </Grid>
</Window>
























