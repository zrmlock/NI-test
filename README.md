# NI-test
measure current change in the process,by C# upper computer

/******************************************************************************
*
* Example program:
*   TdmsAcqVoltageSamples_IntClk
*
* Category:
*   AI
*
* Description:
*   This example demonstrates how to acquire a finite amount while
*   simultaneously streaming that data to a binary file.
*
* Instructions for running:
*   1.  Select the physical channel corresponding to the input signal on the DAQ
*       device.
*   2.  Enter the minimum and maximum voltage ranges.Note:  For better accuracy,
*       try to match the input ranges to the expected voltage level of the
*       measured signal.
*   3.  Select the number of samples per channel to acquire.
*   4.  Set the rate in Hz for the internal clock.Note:  The rate should be at
*       least twice as fast as the maximum frequency component of the signal
*       being acquired.
*   5.  Set the file to write to.
*
* Steps:
*   1.  Create a new task and an analog input voltage channel.
*   2.  Configure the task to use the internal clock.
*   3.  Configure the task to enable TDMS logging.
*   4.  Create a AnalogMultiChannelReader and associate it with the task by
*       using the task's stream.
*   5.  Call AnalogMultiChannelReader.ReadWaveform to acquire and display the
*       data.
*   6.  Dispose the Task object to clean-up any resources associated with the
*       task.
*   7.  Handle any DaqExceptions, if they occur.
*
* I/O Connections Overview:
*   Make sure your signal input terminals match the physical channel text box. 
*   For more information on the input and output terminals for your device, open
*   the NI-DAQmx Help, and refer to the NI-DAQmx Device Terminals and Device
*   Considerations books in the table of contents.
*
* Microsoft Windows Vista User Account Control
*   Running certain applications on Microsoft Windows Vista requires
*   administrator privileges, 
*   because the application name contains keywords such as setup, update, or
*   install. To avoid this problem, 
*   you must add an additional manifest to the application that specifies the
*   privileges required to run 
*   the application. Some Measurement Studio NI-DAQmx examples for Visual Studio
*   include these keywords. 
*   Therefore, all examples for Visual Studio are shipped with an additional
*   manifest file that you must 
*   embed in the example executable. The manifest file is named
*   [ExampleName].exe.manifest, where [ExampleName] 
*   is the NI-provided example name. For information on how to embed the manifest
*   file, refer to http://msdn2.microsoft.com/en-us/library/bb756929.aspx.Note: 
*   The manifest file is not provided with examples for Visual Studio .NET 2003.
*
******************************************************************************/

using System;
using System.Drawing;
using System.Collections;
using System.ComponentModel;
using System.Windows.Forms;
using System.Data;
using NationalInstruments.DAQmx;
using System.IO;
using System.Windows.Forms.DataVisualization.Charting;
using NationalInstruments;
using NationalInstruments.Tdms;

namespace NationalInstruments.Examples.AcqVoltageSamples_IntClk
{
    /// <summary>
    /// Summary description for MainForm.
    /// </summary>
    public class MainForm : System.Windows.Forms.Form
    {        
        private Task myTask;
        private Task runningTask;
        private AnalogMultiChannelReader analogInReader;
        private AsyncCallback analogCallback;
        private AnalogWaveform<double>[] data;
        private DataColumn[] dataColumn = null;
        private DataTable dataTable = null;
        private Series[] series = null;
        private int Channelnumber = 0;
        private int addCH = 0;
        private string deskPath = Environment.GetFolderPath(Environment.SpecialFolder.DesktopDirectory);
        private string foldPath = null;
        private double pointindex = 0;


        private System.Windows.Forms.Button startButton;
        private System.Windows.Forms.GroupBox channelParametersGroupBox;
        private System.Windows.Forms.Label maximumLabel;
        private System.Windows.Forms.Label minimumLabel;
        private System.Windows.Forms.Label physicalChannelLabel;
        private System.Windows.Forms.Label rateLabel;
        private System.Windows.Forms.Label samplesLabel;
        private System.Windows.Forms.GroupBox timingParametersGroupBox;
        private System.Windows.Forms.GroupBox acquisitionResultGroupBox;
        private System.Windows.Forms.NumericUpDown samplesPerChannelNumeric;
        private System.Windows.Forms.DataGrid acquisitionDataGrid;
        private System.Windows.Forms.NumericUpDown rateNumeric;
        internal System.Windows.Forms.NumericUpDown minimumValueNumeric;
        internal System.Windows.Forms.NumericUpDown maximumValueNumeric;
        private System.Windows.Forms.ComboBox physicalChannel0;
        private System.Windows.Forms.ComboBox physicalChannel1;
        private System.Windows.Forms.ComboBox physicalChannel2;
        private System.Windows.Forms.ComboBox physicalChannel3;
        private System.Windows.Forms.ComboBox physicalChannel4;
        private System.Windows.Forms.ComboBox physicalChannel5;
        private ComboBox[] Channelb = new ComboBox[10];

        private GroupBox loggingParametersGroupBox;
        private Label TdmsFilePathLabel;
        private TextBox tdmsFilePathTextBox;
        private System.Windows.Forms.DataVisualization.Charting.Chart chart1;
        private Button stopButton;
        private Button setButton;
        private Button addChannelButton;
        private Button clearChannelButton;
        private Button configureButton;
        private ComboBox weldMaterial1;
        private ComboBox weldMaterial2;
        private Label weldlabel2;
        private Label weldlabel1;
        private GroupBox groupBox1;
        private Button pathButton;
        private TextBox pathTextBox;
        private TextBox textBox1;

        /// <summary>
        /// Required designer variable.
        /// </summary>
        private System.ComponentModel.Container components = null;
        
        public MainForm()
        {
            //
            // Required for Windows Form Designer support
            //
            InitializeComponent();
            newChannel();
            myTask = new Task();
            //
            // TODO: Add any constructor code after InitializeComponent call
            //
            configureButton.Enabled = false;
            startButton.Enabled = false;
            stopButton.Enabled = false;
            dataTable = new DataTable();
            foldPath = deskPath;

            physicalChannel0.Items.AddRange(DaqSystem.Local.GetPhysicalChannels(PhysicalChannelTypes.AI, PhysicalChannelAccess.External));
            if (physicalChannel0.Items.Count > 0)
                physicalChannel0.SelectedIndex = 0;

            weldMaterial1.SelectedIndex = 0;
            weldMaterial2.SelectedIndex = 0;
        }

        /// <summary>
        /// Clean up any resources being used.
        /// </summary>
        protected override void Dispose( bool disposing )
        {
            if (disposing)
            {
                if (components != null)
                {
                    components.Dispose();
                }
                if (myTask != null)
                {
                    runningTask = null;
                    myTask.Dispose();
                }
            }
            base.Dispose(disposing);
        }

        #region Windows Form Designer generated code
        /// <summary>
        /// Required method for Designer support - do not modify
        /// the contents of this method with the code editor.
        /// </summary>
        private void InitializeComponent()
        {
            System.Windows.Forms.DataVisualization.Charting.ChartArea chartArea1 = new System.Windows.Forms.DataVisualization.Charting.ChartArea();
            System.Windows.Forms.DataVisualization.Charting.Legend legend1 = new System.Windows.Forms.DataVisualization.Charting.Legend();
            System.ComponentModel.ComponentResourceManager resources = new System.ComponentModel.ComponentResourceManager(typeof(MainForm));
            this.channelParametersGroupBox = new System.Windows.Forms.GroupBox();
            this.clearChannelButton = new System.Windows.Forms.Button();
            this.addChannelButton = new System.Windows.Forms.Button();
            this.physicalChannel0 = new System.Windows.Forms.ComboBox();
            this.minimumValueNumeric = new System.Windows.Forms.NumericUpDown();
            this.maximumValueNumeric = new System.Windows.Forms.NumericUpDown();
            this.maximumLabel = new System.Windows.Forms.Label();
            this.minimumLabel = new System.Windows.Forms.Label();
            this.physicalChannelLabel = new System.Windows.Forms.Label();
            this.physicalChannel1 = new System.Windows.Forms.ComboBox();
            this.physicalChannel2 = new System.Windows.Forms.ComboBox();
            this.physicalChannel3 = new System.Windows.Forms.ComboBox();
            this.physicalChannel4 = new System.Windows.Forms.ComboBox();
            this.physicalChannel5 = new System.Windows.Forms.ComboBox();
            this.timingParametersGroupBox = new System.Windows.Forms.GroupBox();
            this.rateLabel = new System.Windows.Forms.Label();
            this.samplesLabel = new System.Windows.Forms.Label();
            this.samplesPerChannelNumeric = new System.Windows.Forms.NumericUpDown();
            this.rateNumeric = new System.Windows.Forms.NumericUpDown();
            this.startButton = new System.Windows.Forms.Button();
            this.acquisitionResultGroupBox = new System.Windows.Forms.GroupBox();
            this.acquisitionDataGrid = new System.Windows.Forms.DataGrid();
            this.loggingParametersGroupBox = new System.Windows.Forms.GroupBox();
            this.textBox1 = new System.Windows.Forms.TextBox();
            this.pathTextBox = new System.Windows.Forms.TextBox();
            this.pathButton = new System.Windows.Forms.Button();
            this.setButton = new System.Windows.Forms.Button();
            this.tdmsFilePathTextBox = new System.Windows.Forms.TextBox();
            this.TdmsFilePathLabel = new System.Windows.Forms.Label();
            this.chart1 = new System.Windows.Forms.DataVisualization.Charting.Chart();
            this.stopButton = new System.Windows.Forms.Button();
            this.configureButton = new System.Windows.Forms.Button();
            this.weldMaterial1 = new System.Windows.Forms.ComboBox();
            this.weldMaterial2 = new System.Windows.Forms.ComboBox();
            this.weldlabel2 = new System.Windows.Forms.Label();
            this.weldlabel1 = new System.Windows.Forms.Label();
            this.groupBox1 = new System.Windows.Forms.GroupBox();
            this.channelParametersGroupBox.SuspendLayout();
            ((System.ComponentModel.ISupportInitialize)(this.minimumValueNumeric)).BeginInit();
            ((System.ComponentModel.ISupportInitialize)(this.maximumValueNumeric)).BeginInit();
            this.timingParametersGroupBox.SuspendLayout();
            ((System.ComponentModel.ISupportInitialize)(this.samplesPerChannelNumeric)).BeginInit();
            ((System.ComponentModel.ISupportInitialize)(this.rateNumeric)).BeginInit();
            this.acquisitionResultGroupBox.SuspendLayout();
            ((System.ComponentModel.ISupportInitialize)(this.acquisitionDataGrid)).BeginInit();
            this.loggingParametersGroupBox.SuspendLayout();
            ((System.ComponentModel.ISupportInitialize)(this.chart1)).BeginInit();
            this.groupBox1.SuspendLayout();
            this.SuspendLayout();
            // 
            // channelParametersGroupBox
            // 
            this.channelParametersGroupBox.Controls.Add(this.clearChannelButton);
            this.channelParametersGroupBox.Controls.Add(this.addChannelButton);
            this.channelParametersGroupBox.Controls.Add(this.physicalChannel0);
            this.channelParametersGroupBox.Controls.Add(this.minimumValueNumeric);
            this.channelParametersGroupBox.Controls.Add(this.maximumValueNumeric);
            this.channelParametersGroupBox.Controls.Add(this.maximumLabel);
            this.channelParametersGroupBox.Controls.Add(this.minimumLabel);
            this.channelParametersGroupBox.Controls.Add(this.physicalChannelLabel);
            this.channelParametersGroupBox.FlatStyle = System.Windows.Forms.FlatStyle.System;
            this.channelParametersGroupBox.Location = new System.Drawing.Point(10, 13);
            this.channelParametersGroupBox.Name = "channelParametersGroupBox";
            this.channelParametersGroupBox.Size = new System.Drawing.Size(278, 229);
            this.channelParametersGroupBox.TabIndex = 0;
            this.channelParametersGroupBox.TabStop = false;
            this.channelParametersGroupBox.Text = "Channel Parameters";
            // 
            // clearChannelButton
            // 
            this.clearChannelButton.Location = new System.Drawing.Point(16, 191);
            this.clearChannelButton.Name = "clearChannelButton";
            this.clearChannelButton.Size = new System.Drawing.Size(96, 26);
            this.clearChannelButton.TabIndex = 7;
            this.clearChannelButton.Text = "clear Channel";
            this.clearChannelButton.UseVisualStyleBackColor = true;
            this.clearChannelButton.Click += new System.EventHandler(this.ClearChannelButton_Click);
            // 
            // addChannelButton
            // 
            this.addChannelButton.Location = new System.Drawing.Point(15, 165);
            this.addChannelButton.Name = "addChannelButton";
            this.addChannelButton.Size = new System.Drawing.Size(97, 23);
            this.addChannelButton.TabIndex = 6;
            this.addChannelButton.Text = "add Channel";
            this.addChannelButton.UseVisualStyleBackColor = true;
            this.addChannelButton.Click += new System.EventHandler(this.AddChannelButton_Click);
            // 
            // physicalChannel0
            // 
            this.physicalChannel0.Location = new System.Drawing.Point(144, 20);
            this.physicalChannel0.Name = "physicalChannel0";
            this.physicalChannel0.Size = new System.Drawing.Size(115, 20);
            this.physicalChannel0.TabIndex = 1;
            this.physicalChannel0.Text = "Dev1/ai0";
            // 
            // minimumValueNumeric
            // 
            this.minimumValueNumeric.DecimalPlaces = 3;
            this.minimumValueNumeric.Location = new System.Drawing.Point(6, 81);
            this.minimumValueNumeric.Maximum = new decimal(new int[] {
            24,
            0,
            0,
            196608});
            this.minimumValueNumeric.Name = "minimumValueNumeric";
            this.minimumValueNumeric.Size = new System.Drawing.Size(115, 21);
            this.minimumValueNumeric.TabIndex = 3;
            // 
            // maximumValueNumeric
            // 
            this.maximumValueNumeric.DecimalPlaces = 3;
            this.maximumValueNumeric.Location = new System.Drawing.Point(6, 138);
            this.maximumValueNumeric.Maximum = new decimal(new int[] {
            24,
            0,
            0,
            196608});
            this.maximumValueNumeric.Name = "maximumValueNumeric";
            this.maximumValueNumeric.Size = new System.Drawing.Size(115, 21);
            this.maximumValueNumeric.TabIndex = 5;
            this.maximumValueNumeric.Value = new decimal(new int[] {
            24,
            0,
            0,
            196608});
            // 
            // maximumLabel
            // 
            this.maximumLabel.FlatStyle = System.Windows.Forms.FlatStyle.System;
            this.maximumLabel.Location = new System.Drawing.Point(19, 118);
            this.maximumLabel.Name = "maximumLabel";
            this.maximumLabel.Size = new System.Drawing.Size(115, 17);
            this.maximumLabel.TabIndex = 4;
            this.maximumLabel.Text = "Maximum (Amps):";
            // 
            // minimumLabel
            // 
            this.minimumLabel.FlatStyle = System.Windows.Forms.FlatStyle.System;
            this.minimumLabel.Location = new System.Drawing.Point(19, 60);
            this.minimumLabel.Name = "minimumLabel";
            this.minimumLabel.Size = new System.Drawing.Size(115, 18);
            this.minimumLabel.TabIndex = 2;
            this.minimumLabel.Text = "Minimum (Amps):";
            // 
            // physicalChannelLabel
            // 
            this.physicalChannelLabel.FlatStyle = System.Windows.Forms.FlatStyle.System;
            this.physicalChannelLabel.Location = new System.Drawing.Point(13, 29);
            this.physicalChannelLabel.Name = "physicalChannelLabel";
            this.physicalChannelLabel.Size = new System.Drawing.Size(115, 17);
            this.physicalChannelLabel.TabIndex = 0;
            this.physicalChannelLabel.Text = "Physical Channel:";
            // 
            // physicalChannel1
            // 
            this.physicalChannel1.Location = new System.Drawing.Point(0, 0);
            this.physicalChannel1.Name = "physicalChannel1";
            this.physicalChannel1.Size = new System.Drawing.Size(121, 20);
            this.physicalChannel1.TabIndex = 0;
            // 
            // physicalChannel2
            // 
            this.physicalChannel2.Location = new System.Drawing.Point(0, 0);
            this.physicalChannel2.Name = "physicalChannel2";
            this.physicalChannel2.Size = new System.Drawing.Size(121, 20);
            this.physicalChannel2.TabIndex = 0;
            // 
            // physicalChannel3
            // 
            this.physicalChannel3.Location = new System.Drawing.Point(0, 0);
            this.physicalChannel3.Name = "physicalChannel3";
            this.physicalChannel3.Size = new System.Drawing.Size(121, 20);
            this.physicalChannel3.TabIndex = 0;
            // 
            // physicalChannel4
            // 
            this.physicalChannel4.Location = new System.Drawing.Point(0, 0);
            this.physicalChannel4.Name = "physicalChannel4";
            this.physicalChannel4.Size = new System.Drawing.Size(121, 20);
            this.physicalChannel4.TabIndex = 0;
            // 
            // physicalChannel5
            // 
            this.physicalChannel5.Location = new System.Drawing.Point(0, 0);
            this.physicalChannel5.Name = "physicalChannel5";
            this.physicalChannel5.Size = new System.Drawing.Size(121, 20);
            this.physicalChannel5.TabIndex = 0;
            // 
            // timingParametersGroupBox
            // 
            this.timingParametersGroupBox.Controls.Add(this.rateLabel);
            this.timingParametersGroupBox.Controls.Add(this.samplesLabel);
            this.timingParametersGroupBox.Controls.Add(this.samplesPerChannelNumeric);
            this.timingParametersGroupBox.Controls.Add(this.rateNumeric);
            this.timingParametersGroupBox.FlatStyle = System.Windows.Forms.FlatStyle.System;
            this.timingParametersGroupBox.Location = new System.Drawing.Point(10, 248);
            this.timingParametersGroupBox.Name = "timingParametersGroupBox";
            this.timingParametersGroupBox.Size = new System.Drawing.Size(278, 95);
            this.timingParametersGroupBox.TabIndex = 1;
            this.timingParametersGroupBox.TabStop = false;
            this.timingParametersGroupBox.Text = "Timing Parameters";
            // 
            // rateLabel
            // 
            this.rateLabel.FlatStyle = System.Windows.Forms.FlatStyle.System;
            this.rateLabel.Location = new System.Drawing.Point(19, 60);
            this.rateLabel.Name = "rateLabel";
            this.rateLabel.Size = new System.Drawing.Size(77, 18);
            this.rateLabel.TabIndex = 2;
            this.rateLabel.Text = "Rate (Hz):";
            // 
            // samplesLabel
            // 
            this.samplesLabel.FlatStyle = System.Windows.Forms.FlatStyle.System;
            this.samplesLabel.Location = new System.Drawing.Point(19, 26);
            this.samplesLabel.Name = "samplesLabel";
            this.samplesLabel.Size = new System.Drawing.Size(125, 17);
            this.samplesLabel.TabIndex = 0;
            this.samplesLabel.Text = "Samples / Channel:";
            // 
            // samplesPerChannelNumeric
            // 
            this.samplesPerChannelNumeric.Location = new System.Drawing.Point(144, 26);
            this.samplesPerChannelNumeric.Maximum = new decimal(new int[] {
            100000,
            0,
            0,
            0});
            this.samplesPerChannelNumeric.Name = "samplesPerChannelNumeric";
            this.samplesPerChannelNumeric.Size = new System.Drawing.Size(115, 21);
            this.samplesPerChannelNumeric.TabIndex = 1;
            this.samplesPerChannelNumeric.Value = new decimal(new int[] {
            1000,
            0,
            0,
            0});
            // 
            // rateNumeric
            // 
            this.rateNumeric.DecimalPlaces = 2;
            this.rateNumeric.Location = new System.Drawing.Point(144, 60);
            this.rateNumeric.Maximum = new decimal(new int[] {
            100000,
            0,
            0,
            0});
            this.rateNumeric.Name = "rateNumeric";
            this.rateNumeric.Size = new System.Drawing.Size(115, 21);
            this.rateNumeric.TabIndex = 3;
            this.rateNumeric.Value = new decimal(new int[] {
            1000,
            0,
            0,
            0});
            // 
            // startButton
            // 
            this.startButton.FlatStyle = System.Windows.Forms.FlatStyle.System;
            this.startButton.Location = new System.Drawing.Point(368, 29);
            this.startButton.Name = "startButton";
            this.startButton.Size = new System.Drawing.Size(96, 26);
            this.startButton.TabIndex = 3;
            this.startButton.Text = "Start";
            this.startButton.Click += new System.EventHandler(this.startButton_Click);
            // 
            // acquisitionResultGroupBox
            // 
            this.acquisitionResultGroupBox.Controls.Add(this.acquisitionDataGrid);
            this.acquisitionResultGroupBox.FlatStyle = System.Windows.Forms.FlatStyle.System;
            this.acquisitionResultGroupBox.Location = new System.Drawing.Point(300, 13);
            this.acquisitionResultGroupBox.Name = "acquisitionResultGroupBox";
            this.acquisitionResultGroupBox.Size = new System.Drawing.Size(573, 16);
            this.acquisitionResultGroupBox.TabIndex = 4;
            this.acquisitionResultGroupBox.TabStop = false;
            this.acquisitionResultGroupBox.Text = "Acquisition Results";
            // 
            // acquisitionDataGrid
            // 
            this.acquisitionDataGrid.AllowSorting = false;
            this.acquisitionDataGrid.DataMember = "";
            this.acquisitionDataGrid.HeaderForeColor = System.Drawing.SystemColors.ControlText;
            this.acquisitionDataGrid.Location = new System.Drawing.Point(6, 20);
            this.acquisitionDataGrid.Name = "acquisitionDataGrid";
            this.acquisitionDataGrid.ParentRowsVisible = false;
            this.acquisitionDataGrid.ReadOnly = true;
            this.acquisitionDataGrid.Size = new System.Drawing.Size(409, 327);
            this.acquisitionDataGrid.TabIndex = 1;
            this.acquisitionDataGrid.TabStop = false;
            // 
            // loggingParametersGroupBox
            // 
            this.loggingParametersGroupBox.Controls.Add(this.textBox1);
            this.loggingParametersGroupBox.Controls.Add(this.pathTextBox);
            this.loggingParametersGroupBox.Controls.Add(this.pathButton);
            this.loggingParametersGroupBox.Controls.Add(this.setButton);
            this.loggingParametersGroupBox.Controls.Add(this.tdmsFilePathTextBox);
            this.loggingParametersGroupBox.Controls.Add(this.TdmsFilePathLabel);
            this.loggingParametersGroupBox.Location = new System.Drawing.Point(10, 349);
            this.loggingParametersGroupBox.Name = "loggingParametersGroupBox";
            this.loggingParametersGroupBox.Size = new System.Drawing.Size(278, 119);
            this.loggingParametersGroupBox.TabIndex = 2;
            this.loggingParametersGroupBox.TabStop = false;
            this.loggingParametersGroupBox.Text = "Logging Parameters";
            // 
            // textBox1
            // 
            this.textBox1.BorderStyle = System.Windows.Forms.BorderStyle.None;
            this.textBox1.Font = new System.Drawing.Font("宋体", 10.5F, System.Drawing.FontStyle.Regular, System.Drawing.GraphicsUnit.Point, ((byte)(134)));
            this.textBox1.Location = new System.Drawing.Point(6, 73);
            this.textBox1.Multiline = true;
            this.textBox1.Name = "textBox1";
            this.textBox1.ReadOnly = true;
            this.textBox1.Size = new System.Drawing.Size(50, 40);
            this.textBox1.TabIndex = 5;
            this.textBox1.Text = "文件保存信息";
            // 
            // pathTextBox
            // 
            this.pathTextBox.BorderStyle = System.Windows.Forms.BorderStyle.None;
            this.pathTextBox.Location = new System.Drawing.Point(74, 71);
            this.pathTextBox.Multiline = true;
            this.pathTextBox.Name = "pathTextBox";
            this.pathTextBox.ReadOnly = true;
            this.pathTextBox.Size = new System.Drawing.Size(191, 42);
            this.pathTextBox.TabIndex = 4;
            this.pathTextBox.TabStop = false;
            // 
            // pathButton
            // 
            this.pathButton.Location = new System.Drawing.Point(124, 12);
            this.pathButton.Name = "pathButton";
            this.pathButton.Size = new System.Drawing.Size(33, 26);
            this.pathButton.TabIndex = 3;
            this.pathButton.Text = "...";
            this.pathButton.UseVisualStyleBackColor = true;
            this.pathButton.Click += new System.EventHandler(this.PathButton_Click);
            // 
            // setButton
            // 
            this.setButton.Location = new System.Drawing.Point(163, 12);
            this.setButton.Name = "setButton";
            this.setButton.Size = new System.Drawing.Size(96, 26);
            this.setButton.TabIndex = 2;
            this.setButton.Text = "set TDMS";
            this.setButton.UseVisualStyleBackColor = true;
            this.setButton.Click += new System.EventHandler(this.SetButton_Click);
            // 
            // tdmsFilePathTextBox
            // 
            this.tdmsFilePathTextBox.Location = new System.Drawing.Point(23, 44);
            this.tdmsFilePathTextBox.Name = "tdmsFilePathTextBox";
            this.tdmsFilePathTextBox.Size = new System.Drawing.Size(236, 21);
            this.tdmsFilePathTextBox.TabIndex = 1;
            // 
            // TdmsFilePathLabel
            // 
            this.TdmsFilePathLabel.AutoSize = true;
            this.TdmsFilePathLabel.Location = new System.Drawing.Point(19, 26);
            this.TdmsFilePathLabel.Name = "TdmsFilePathLabel";
            this.TdmsFilePathLabel.Size = new System.Drawing.Size(95, 12);
            this.TdmsFilePathLabel.TabIndex = 0;
            this.TdmsFilePathLabel.Text = "TDMS File Path:";
            // 
            // chart1
            // 
            chartArea1.Name = "ChartArea1";
            this.chart1.ChartAreas.Add(chartArea1);
            legend1.Name = "电流";
            this.chart1.Legends.Add(legend1);
            this.chart1.Location = new System.Drawing.Point(300, 27);
            this.chart1.Name = "chart1";
            this.chart1.Size = new System.Drawing.Size(575, 376);
            this.chart1.TabIndex = 5;
            this.chart1.Text = "chart1";
            // 
            // stopButton
            // 
            this.stopButton.Location = new System.Drawing.Point(483, 29);
            this.stopButton.Name = "stopButton";
            this.stopButton.Size = new System.Drawing.Size(96, 26);
            this.stopButton.TabIndex = 6;
            this.stopButton.Text = "Stop";
            this.stopButton.UseVisualStyleBackColor = true;
            this.stopButton.Click += new System.EventHandler(this.StopButton_Click);
            // 
            // configureButton
            // 
            this.configureButton.Location = new System.Drawing.Point(257, 29);
            this.configureButton.Name = "configureButton";
            this.configureButton.Size = new System.Drawing.Size(96, 26);
            this.configureButton.TabIndex = 8;
            this.configureButton.Text = "Configure";
            this.configureButton.UseVisualStyleBackColor = true;
            this.configureButton.Click += new System.EventHandler(this.ConfigureButton_Click);
            // 
            // weldMaterial1
            // 
            this.weldMaterial1.FormattingEnabled = true;
            this.weldMaterial1.Items.AddRange(new object[] {
            "铝及其合金",
            "钢",
            "铜及其合金",
            "镁及其合金",
            "钛及其合金",
            "镍及其合金"});
            this.weldMaterial1.Location = new System.Drawing.Point(12, 35);
            this.weldMaterial1.Name = "weldMaterial1";
            this.weldMaterial1.Size = new System.Drawing.Size(96, 20);
            this.weldMaterial1.TabIndex = 9;
            // 
            // weldMaterial2
            // 
            this.weldMaterial2.FormattingEnabled = true;
            this.weldMaterial2.Items.AddRange(new object[] {
            "铝及其合金",
            "钢",
            "铜及其合金",
            "镁及其合金",
            "钛及其合金",
            "镍及其合金"});
            this.weldMaterial2.Location = new System.Drawing.Point(128, 35);
            this.weldMaterial2.Name = "weldMaterial2";
            this.weldMaterial2.Size = new System.Drawing.Size(96, 20);
            this.weldMaterial2.TabIndex = 10;
            // 
            // weldlabel2
            // 
            this.weldlabel2.AutoSize = true;
            this.weldlabel2.Font = new System.Drawing.Font("宋体", 9F, System.Drawing.FontStyle.Regular, System.Drawing.GraphicsUnit.Point, ((byte)(134)));
            this.weldlabel2.Location = new System.Drawing.Point(126, 20);
            this.weldlabel2.Name = "weldlabel2";
            this.weldlabel2.Size = new System.Drawing.Size(41, 12);
            this.weldlabel2.TabIndex = 11;
            this.weldlabel2.Text = "材料二";
            // 
            // weldlabel1
            // 
            this.weldlabel1.AutoSize = true;
            this.weldlabel1.Font = new System.Drawing.Font("宋体", 9F, System.Drawing.FontStyle.Regular, System.Drawing.GraphicsUnit.Point, ((byte)(134)));
            this.weldlabel1.Location = new System.Drawing.Point(10, 20);
            this.weldlabel1.Name = "weldlabel1";
            this.weldlabel1.Size = new System.Drawing.Size(41, 12);
            this.weldlabel1.TabIndex = 12;
            this.weldlabel1.Text = "材料一";
            // 
            // groupBox1
            // 
            this.groupBox1.Controls.Add(this.weldMaterial2);
            this.groupBox1.Controls.Add(this.weldlabel1);
            this.groupBox1.Controls.Add(this.startButton);
            this.groupBox1.Controls.Add(this.weldlabel2);
            this.groupBox1.Controls.Add(this.stopButton);
            this.groupBox1.Controls.Add(this.configureButton);
            this.groupBox1.Controls.Add(this.weldMaterial1);
            this.groupBox1.Location = new System.Drawing.Point(294, 409);
            this.groupBox1.Name = "groupBox1";
            this.groupBox1.Size = new System.Drawing.Size(599, 67);
            this.groupBox1.TabIndex = 13;
            this.groupBox1.TabStop = false;
            this.groupBox1.Text = "Measure Group";
            // 
            // MainForm
            // 
            this.AutoScaleBaseSize = new System.Drawing.Size(6, 14);
            this.ClientSize = new System.Drawing.Size(905, 480);
            this.Controls.Add(this.groupBox1);
            this.Controls.Add(this.chart1);
            this.Controls.Add(this.loggingParametersGroupBox);
            this.Controls.Add(this.acquisitionResultGroupBox);
            this.Controls.Add(this.timingParametersGroupBox);
            this.Controls.Add(this.channelParametersGroupBox);
            this.Icon = ((System.Drawing.Icon)(resources.GetObject("$this.Icon")));
            this.MaximizeBox = false;
            this.Name = "MainForm";
            this.StartPosition = System.Windows.Forms.FormStartPosition.CenterScreen;
            this.Text = "TDMS Current Measure";
            this.Load += new System.EventHandler(this.MainForm_Load);
            this.channelParametersGroupBox.ResumeLayout(false);
            ((System.ComponentModel.ISupportInitialize)(this.minimumValueNumeric)).EndInit();
            ((System.ComponentModel.ISupportInitialize)(this.maximumValueNumeric)).EndInit();
            this.timingParametersGroupBox.ResumeLayout(false);
            ((System.ComponentModel.ISupportInitialize)(this.samplesPerChannelNumeric)).EndInit();
            ((System.ComponentModel.ISupportInitialize)(this.rateNumeric)).EndInit();
            this.acquisitionResultGroupBox.ResumeLayout(false);
            ((System.ComponentModel.ISupportInitialize)(this.acquisitionDataGrid)).EndInit();
            this.loggingParametersGroupBox.ResumeLayout(false);
            this.loggingParametersGroupBox.PerformLayout();
            ((System.ComponentModel.ISupportInitialize)(this.chart1)).EndInit();
            this.groupBox1.ResumeLayout(false);
            this.groupBox1.PerformLayout();
            this.ResumeLayout(false);

        }
        #endregion

        /// <summary>
        /// The main entry point for the application.
        /// </summary>
        [STAThread]
        static void Main() 
        {
            Application.EnableVisualStyles();
            Application.DoEvents();
            Application.Run(new MainForm());
        }
        private void newChannel()
        {
            Channelnumber = 0;
            Channelb[Channelnumber] = physicalChannel0;

            Channelnumber = 1;
            string comboxName = "physicalChannel" + Channelnumber;
            physicalChannel1.Size = new Size(115, 20);
            physicalChannel1.Name = comboxName;
            physicalChannel1.Location = new Point(144, 35 * Channelnumber + 20);
            physicalChannel1.Items.AddRange(DaqSystem.Local.GetPhysicalChannels(PhysicalChannelTypes.AI, PhysicalChannelAccess.External));
            Channelb[Channelnumber] = physicalChannel1;

            Channelnumber += 1;
            comboxName = "physicalChannel" + Channelnumber;
            physicalChannel2.Size = new Size(115, 20);
            physicalChannel2.Name = comboxName;
            physicalChannel2.Location = new Point(144, 35 * Channelnumber + 20);
            physicalChannel2.Items.AddRange(DaqSystem.Local.GetPhysicalChannels(PhysicalChannelTypes.AI, PhysicalChannelAccess.External));
            Channelb[Channelnumber] = physicalChannel2;

            Channelnumber += 1;
            comboxName = "physicalChannel" + Channelnumber;
            physicalChannel3.Size = new Size(115, 20);
            physicalChannel3.Name = comboxName;
            physicalChannel3.Location = new Point(144, 35 * Channelnumber + 20);
            physicalChannel3.Items.AddRange(DaqSystem.Local.GetPhysicalChannels(PhysicalChannelTypes.AI, PhysicalChannelAccess.External));
            Channelb[Channelnumber] = physicalChannel3;

            Channelnumber += 1;
            comboxName = "physicalChannel" + Channelnumber;
            physicalChannel4.Size = new Size(115, 20);
            physicalChannel4.Name = comboxName;
            physicalChannel4.Location = new Point(144, 35 * Channelnumber + 20);
            physicalChannel4.Items.AddRange(DaqSystem.Local.GetPhysicalChannels(PhysicalChannelTypes.AI, PhysicalChannelAccess.External));
            Channelb[Channelnumber] = physicalChannel4;

            Channelnumber += 1;
            comboxName = "physicalChannel" + Channelnumber;
            physicalChannel5.Size = new Size(115, 20);
            physicalChannel5.Name = comboxName;
            physicalChannel5.Location = new Point(144, 35 * Channelnumber + 20);
            physicalChannel5.Items.AddRange(DaqSystem.Local.GetPhysicalChannels(PhysicalChannelTypes.AI, PhysicalChannelAccess.External));
            Channelb[Channelnumber] = physicalChannel5;
    }
        private void startButton_Click(object sender, System.EventArgs e)
        {
            if (runningTask == null)
            {
                try
                {
                    stopButton.Enabled = true;
                    startButton.Enabled = false;
                    pointindex = 0;
                    //准备数据表
                    InitializeDataTable(myTask.AIChannels, ref dataTable);
                    InitializeChart(myTask.AIChannels, ref chart1);
                    acquisitionDataGrid.DataSource = dataTable;
                    chart1.DataSource = dataTable;
                    //chart1.Series.

                    runningTask = myTask;
                    analogInReader = new AnalogMultiChannelReader(myTask.Stream);
                    analogCallback = new AsyncCallback(AnalogInCallback);

                    analogInReader.SynchronizeCallbacks = true;
                    analogInReader.BeginReadWaveform(Convert.ToInt32(samplesPerChannelNumeric.Value),
                        analogCallback, myTask);
                }
                catch (DaqException exception)
                {
                    MessageBox.Show(exception.Message);
                    runningTask = null;
                    myTask.Dispose();
                    stopButton.Enabled = false;
                    startButton.Enabled = true;
                }
            }
        }
        private void AnalogInCallback(IAsyncResult ar)
        {
            try
            {
                if (runningTask != null && runningTask == ar.AsyncState)
                {
                    //异步停止并读取数据
                    data = analogInReader.EndReadWaveform(ar);

                    //绘制数据
                    dataToDataTable(data, ref dataTable);

                    analogInReader.BeginMemoryOptimizedReadWaveform(Convert.ToInt32(samplesPerChannelNumeric.Value),
                        analogCallback, myTask, data);
                }
            }
            catch (DaqException exception)
            {
                MessageBox.Show(exception.Message);
                runningTask = null;
                myTask.Dispose();
                stopButton.Enabled = false;
                startButton.Enabled = true;
            }
        }

        private void dataToDataTable(AnalogWaveform<double>[] sourceArray, ref DataTable dataTable)
        {
            // Iterate over channels
            int currentLineIndex = 0;
            foreach (AnalogWaveform<double> waveform in sourceArray)
            {
                for (int sample = 0; sample < waveform.Samples.Count; ++sample)
                {
                    if (sample == 999)
                        break;

                    dataTable.Rows[sample][currentLineIndex] = waveform.Samples[sample].Value;
                    //chart1.Series[]
                    chart1.Series[currentLineIndex].Points.AddXY(pointindex, waveform.Samples[sample].Value*59375-87.5);
                }
                currentLineIndex++;   
            }
            pointindex += 1;
        }

        public void InitializeDataTable(AIChannelCollection channelCollection, ref DataTable data)
        {
            int numOfChannels = channelCollection.Count;
            data.Rows.Clear();
            data.Columns.Clear();
            dataColumn = new DataColumn[numOfChannels];
            int numOfRows = 999;

            for (int currentChannelIndex = 0; currentChannelIndex < numOfChannels; currentChannelIndex++)
            {   
                dataColumn[currentChannelIndex] = new DataColumn();
                dataColumn[currentChannelIndex].DataType = typeof(double);
                dataColumn[currentChannelIndex].ColumnName = channelCollection[currentChannelIndex].PhysicalName;
            }

            data.Columns.AddRange(dataColumn); 

            for (int currentDataIndex = 0; currentDataIndex < numOfRows; currentDataIndex++)             
            {
                object[] rowArr = new object[numOfChannels];
                data.Rows.Add(rowArr);              
            }
        }

        public void InitializeChart(AIChannelCollection channelCollection,ref Chart chart)
        {
            string[] strcolor = new string[20];
            strcolor[0] = "220, 224, 64, 10";
            strcolor[1] = "220, 252, 180, 65";
            strcolor[2] = "220, 159, 100, 100";
            strcolor[3] = "220, 5, 100, 146";
            strcolor[4] = "91,42,0";
            strcolor[5] = "19,211,188";
            strcolor[6] = "0,93,70";
            strcolor[7] = "185,147,240";
            strcolor[8] = "194,211,252";
            strcolor[9] = "49,0,93";
            strcolor[10] = "245,111,5";
            strcolor[11] = "203,72,178";
            strcolor[12] = "93,93,0";
            strcolor[13] = "165,165,147";
            strcolor[14] = "124,201,15";
            strcolor[15] = "14,112,201";
            strcolor[16] = "0,59,93";
            strcolor[17] = "5,18,108";
            strcolor[18] = "245,15,54";
            strcolor[19] = "121,129,234";
            chart1.Series.Clear();

            int numOfChannels = channelCollection.Count;
            for (int currentChannelIndex = 0; currentChannelIndex < numOfChannels; currentChannelIndex++)
            {
                string name = "sery" + currentChannelIndex;
                chart1.Series.Add(name);
                chart1.Series[name].ChartType = SeriesChartType.Line;
                chart1.Series[name].IsXValueIndexed = true;
                chart1.Series[name].LegendText = name;
                chart1.Series[name].BorderColor = Color.FromArgb(180, 26, 59, 105);

                string[] number = strcolor[currentChannelIndex].ToString().Split(new Char[] { ',' });
                int alpha = int.Parse(number[0].ToString());
                int red = int.Parse(number[1].ToString());
                int green = int.Parse(number[2].ToString());
                int blue = int.Parse(number[3].ToString());
                chart1.Series[name].Color = Color.FromArgb(alpha, red, green, blue);
            }
        }
        
        private void MainForm_Load(object sender, EventArgs e)
        {

        }

        private void StopButton_Click(object sender, EventArgs e)
        {
            if (runningTask != null)
            {
                runningTask = null;
                myTask.Dispose();
                tdmsFilePathTextBox.Clear();
                stopButton.Enabled = false;
                startButton.Enabled = false;
                configureButton.Enabled = false;
                setButton.Enabled = true;
            }
        }

        private void SetButton_Click(object sender, EventArgs e)
        {
            if (File.Exists(deskPath + "\\" + tdmsFilePathTextBox.Text))
            {
                File.Delete(deskPath + "\\" + tdmsFilePathTextBox.Text);
            }
            if (tdmsFilePathTextBox.Text == "")
            {
                tdmsFilePathTextBox.Text = DateTime.Now.ToString("yyyy-MM-dd-HH-mm-ss");
            }

            configureButton.Enabled = true;
        }

        private void AddChannelButton_Click(object sender, EventArgs e)
        {
            addCH += 1;
            this.channelParametersGroupBox.Controls.Add(Channelb[addCH]);
        }

        private void ClearChannelButton_Click(object sender, EventArgs e)
        {
            for (int Channelcount = 1; Channelcount <= addCH; Channelcount++)
            {
                this.channelParametersGroupBox.Controls.Remove(Channelb[Channelcount]);
            }
            addCH = 0;
            myTask.Dispose();
            myTask = new Task();
        }

        private void ConfigureButton_Click(object sender, EventArgs e)
        {
            chart1.Series.Clear();
            if (myTask == null)
                myTask = new Task();
            else
            {
                myTask.Dispose();
                myTask = new Task();
            }
            if (tdmsFilePathTextBox.Text.Trim().Length > 0)
            {
                pathTextBox.Text = (foldPath + "\\" + weldMaterial1.Text + "与" + weldMaterial2.Text + tdmsFilePathTextBox.Text).ToString();
                try
                {
                    for (int n = 0; n <= addCH; n++)//配置通道
                    {
                        myTask.AIChannels.CreateCurrentChannel(Channelb[n].Text, "",
                                (AITerminalConfiguration)(-1), Convert.ToDouble(minimumValueNumeric.Value),
                                Convert.ToDouble(maximumValueNumeric.Value), AICurrentUnits.Amps);
                    }
                    //配置定时器
                    myTask.Timing.ConfigureSampleClock("", Convert.ToDouble(rateNumeric.Value),
                        SampleClockActiveEdge.Rising, SampleQuantityMode.ContinuousSamples, 1000);

                    //配置TDMS
                    myTask.ConfigureLogging(foldPath + "\\" +weldMaterial1.Text + "与" + weldMaterial2.Text + tdmsFilePathTextBox.Text, TdmsLoggingOperation.CreateOrReplace, LoggingMode.LogAndRead, "Group Name");

                    //验证
                    myTask.Control(TaskAction.Verify);
                    startButton.Enabled = true;
                }
                catch (DaqException exception)
                {
                    MessageBox.Show(exception.Message);
                    myTask.Dispose();
                    tdmsFilePathTextBox.Clear();
                    configureButton.Enabled = false;
                }
            }
        }

        private void PathButton_Click(object sender, EventArgs e)//配置保存路径
        {
            FolderBrowserDialog dialog = new FolderBrowserDialog();
            dialog.Description = "选择文件路径";

            if (dialog.ShowDialog() == DialogResult.OK)
            {
                foldPath = dialog.SelectedPath;
                DirectoryInfo theFolder = new DirectoryInfo(foldPath);

                //theFolder 包含文件路径
                //遍历文件夹
                /*FileInfo[] dirInfo = theFolder.GetFiles();                               
                foreach (FileInfo file in dirInfo)
                {
                    MessageBox.Show(file.ToString());
                }*/
            }
        }
    }
}
