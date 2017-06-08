# ChecadorCsharp

using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.IO.Ports;
using System.Data.SQLite;
using System.IO;



namespace WindowsFormsApp1
{

    public partial class Form1 : Form
    {
        string db;
        string connString;
        string query;
        bool isConnected = false;
        String[] ports;
        SerialPort port = new SerialPort();
        

        public Form1()
        {
            InitializeComponent();
            string db = Environment.GetFolderPath(Environment.SpecialFolder.Desktop) + @"\UsuariosGarage";
            string connString = "Data Source=" + db + ";Version=3;";
            if(!File.Exists(db))
            {
                SQLiteConnection.CreateFile(db);
                SQLiteConnection conn = new SQLiteConnection(connString);
                conn.Open();
                SQLiteCommand cmd = new SQLiteCommand("CREATE TABLE Usuarios ( `id` INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT, `RFID` INTEGER NOT NULL, `Nombre` TEXT NOT NULL, `Apellido` TEXT NOT NULL, `IN` TEXT NOT NULL, `OUT` TEXT NOT NULL, `Date` TEXT NOT NULL )");
                cmd.ExecuteNonQuery();
                MessageBox.Show("Se creo la base de datos");
                conn.Close();
                conn.Dispose();
                FillTable();
            }
            disableControls();
            getAvailableComPorts();
            foreach (string port in ports)
            {
                comboBox1.Items.Add(port);
                Console.WriteLine(port);
                if(ports[0] != null)
                {
                    comboBox1.SelectedItem = ports[0];
                }
            }
            
        }
        private void update()
        {
            try
            {
                DialogResult simon = MessageBox.Show("Seguro que desea enviar?", "Atención", MessageBoxButtons.YesNo, MessageBoxIcon.Question);
                if (simon == DialogResult.Yes)
                {
                    string db = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments) + @"\Cardenas.sqlite";
                    string connString = "Data Source=" + db + ";Version=3;";

                    query = "INSERT INTO Usuarios (id,RFID,Nombre, Apellido,IN,OUT,Date) VALUES ('" + textBox3.Text + "','" + textBox1.Text + "','" + textBox2 +"','" + DateTime.Now.ToString("HH:mm:ss") +"','"+DateTime.Now.ToString("yyyy-MM-dd")+ "')";
                    SQLiteConnection conn = new SQLiteConnection(connString);
                    conn.Open();
                    SQLiteCommand cmd = new SQLiteCommand(query, conn);
                    MessageBox.Show("LLEGUE AQUI");
                    cmd.ExecuteNonQuery();
                    conn.Close();
                    conn.Dispose();
                }
                else
                {
                    MessageBox.Show("NO SE ENVIO");
                }


            }
            catch (Exception ex)
            {
                MessageBox.Show("ERROR " + ex.StackTrace + " " + ex.Message);
            }
        }
        void FillTable()
        {
            string db = Environment.GetFolderPath(Environment.SpecialFolder.Desktop) + @"\UsuariosGarage";
            string connString = "Data Source= " + db + ";Version3;";
            SQLiteConnection conn = new SQLiteConnection(connString);
            conn.Open();
            SQLiteDataAdapter cmd = new SQLiteDataAdapter(" Select * from Usuarios ORDER BY Fecha desc limit 1", conn);
            DataSet ds = new DataSet();
            cmd.Fill(ds);
            DataView source = new DataView(ds.Tables[0]);
            dataGridView1.DataSource = source;


        }
        void getAvailableComPorts()
        {
            ports = SerialPort.GetPortNames();
            foreach (string port in ports)
            {
                comboBox1.Items.Add(port);
                Console.WriteLine(port);
                if(ports[0]!=null)
                {
                    comboBox1.SelectedItem = ports[0];
                }
            }

        }
        void connectToArduino()
        {
            isConnected = true;
            string selectedPort = comboBox1.GetItemText(comboBox1.SelectedItem);
            port = new SerialPort(selectedPort, 9600, Parity.None, 8, StopBits.One);
            port.Open();
            port.Write("#START\n");
            Conectar.Text = "Disconnect";
            enableControls();
        }
        private void disconnectFromArduino()
        {
            isConnected = false;
            port.Write("#STOP\n");
            port.Close();
            Conectar.Text = "Connect";
            disableControls();
        }
         private void enableControls()
        {
            textBox1.Enabled = true;
            textBox2.Enabled = true;
            textBox3.Enabled = true;
            Scan.Enabled = true;
            Agregar.Enabled = true;
           
        }
        private void disableControls()
        {
            textBox1.Enabled = false;
            textBox2.Enabled = false;
            textBox3.Enabled = false;
            Scan.Enabled = false;
            Agregar.Enabled = false;

        }
        private void panel1_Paint(object sender, PaintEventArgs e)
        {

        }

     
        private void button1_Click(object sender, EventArgs e)
        {
            //Para Enviar los Textbox a la base de datos



        }

        private void button2_Click(object sender, EventArgs e)
        {
            string rfid ="";
                if (isConnected)
            {
                port.Write("#C");
                rfid = port.ReadLine();
                textBox3.Text = rfid;
            } else {
                //Mensaje no se pudo Escanear RFID
               
            }
            
        }

        private void button3_Click(object sender, EventArgs e)
        {
            //Conectando arduino
            if (!isConnected)
            {
                connectToArduino();
            }else
            {
                disconnectFromArduino();
            }
            }

        private void comboBox1_DropDown(object sender, EventArgs e)
        {
            getAvailableComPorts();
        }
    }
    }
    

