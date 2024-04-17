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












