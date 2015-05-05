using System;
using System.IO;
using System.Drawing.Imaging;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Windows.Forms;

namespace HW1
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            Initialize();
        }
        void Initialize()
        {
            this.ClientSize = new System.Drawing.Size(1200, 600); //視窗大小
            this.AutoSizeMode = AutoSizeMode.GrowAndShrink; //鎖定大小
            this.MaximizeBox = false; //不可放大
            //this.Opacity = 0.95; //透明度
            this.Name = "form";
            this.Text = "Image Processing";

            new_form(); //新增彈出視窗

            new_box(520, 580); //新增 picture box
            new_panel(520, 580); //加入 panel
            this.Controls.Add(this.panel1);
            this.Controls.Add(this.panel2);

            new_menu(); //新增 menu
            this.Menu = menu; //加入 menu

            new_button(8, 1200, 20); //新增按鈕
            new_group(1200); //新增群組

            this.button[0].Text = "RGB extraction";
            this.button[0].Click += new System.EventHandler(this.rgb_extraction);
            this.button[0].Location = new System.Drawing.Point(0, 35);
            this.group.Controls.Add(this.button[0]); //按鈕加入群組
            this.Controls.Add(this.group); //群組加入視窗

            this.button[1].Text = "Grayscale";
            this.button[1].Click += new EventHandler(this.grayscale);
            this.Controls.Add(this.button[1]); //按鈕加入視窗

            new_threshold(1200);
            this.Controls.Add(this.threshold); //視窗加入checkbox
            this.Controls.Add(this.bar); //視窗加入bar

            this.button[3].Text = "Histogram Equalization";
            this.button[3].Location = new System.Drawing.Point(this.button[3].Location.X, this.button[3].Location.Y + 5);
            this.button[3].Click += new EventHandler(histogram_equalize);
            this.Controls.Add(this.button[3]);

            new_edge(1200);
            this.Controls.Add(this.vertical);
            this.Controls.Add(this.horizontal);

            this.color = Color.Red;
            this.button[5].Text = "Overlap";
            this.button[5].Click += new EventHandler(overlap);
            this.Controls.Add(this.button[5]);

            this.button[6].Text = "Save Step";
            this.button[6].Click += new EventHandler(save_step);
            this.Controls.Add(this.button[6]);

            this.button[7].Text = "Reset All";
            this.button[7].Click += new EventHandler(this.reset);
            this.Controls.Add(this.button[7]);

            this.FormClosing+=new FormClosingEventHandler(Image_Processing_close);
        }
        private void Image_Processing_close(object sender, FormClosingEventArgs e)
        {
            if (MessageBox.Show("Are you sure?\nYou will miss unsave data.", "colse", MessageBoxButtons.YesNo) == DialogResult.No) e.Cancel = true;
            else
            {
                /*釋放資源*/
                Choose_range.Dispose();
                this.Dispose();
            }
            return;
        }

        private void new_form()
        {
            Choose_range.ClientSize = new Size(200 ,200);
            Choose_range.Opacity = 0.80; //透明度

            Gamma.Text = Convert.ToString(this.gamma);
            Gamma.Size = new System.Drawing.Size(160, 10);
            Gamma.Location = new Point(20, 20);
            Gamma.KeyPress+=new KeyPressEventHandler(only_num);

            choose_min.Size = new System.Drawing.Size(160, 10);
            choose_min.Location = new Point(20, 45);
            choose_min.Minimum = 0;
            choose_min.Maximum = this.max - 1;
            choose_min.Value = 0;
            choose_min.TickStyle = TickStyle.BottomRight; //刻度位於下方
            choose_min.TickFrequency = 5; //刻度間隔
            choose_min.LargeChange = 10; //按兩側 移動刻度
            choose_min.SmallChange = 1; //用鍵盤方向鍵 移動刻度
            choose_min.ValueChanged+=new EventHandler(RangeChange);

            choose_max.Size = new System.Drawing.Size(160, 10);
            choose_max.Location = new Point(20, 100);
            choose_max.Minimum = this.min + 1;
            choose_max.Maximum = 255;
            choose_max.Value = 255;
            choose_max.TickStyle = TickStyle.BottomRight; //刻度位於下方
            choose_max.TickFrequency = 5; //刻度間隔
            choose_max.LargeChange = 10; //按兩側 移動刻度
            choose_max.SmallChange = 1; //用鍵盤方向鍵 移動刻度
            choose_max.ValueChanged += new EventHandler(RangeChange);

            showrange.Size = new System.Drawing.Size(80, 20);
            showrange.Location = new Point(20, 160);
            showrange.Text = "Range:" + min.ToString() + "~" + max.ToString();

            Yes_range.Size = new System.Drawing.Size(80, 30);
            Yes_range.Location = new Point(100, 150);
            Yes_range.Text = "OK";
            Yes_range.Click+=new EventHandler(Yes_range_Click);

            Choose_range.Controls.Add(Gamma);
            Choose_range.Controls.Add(choose_min);
            Choose_range.Controls.Add(choose_max);
            Choose_range.Controls.Add(showrange);
            Choose_range.Controls.Add(Yes_range);
        }
        private void only_num(object sender, KeyPressEventArgs e)
        {
            if ((int)e.KeyChar >= 48 || (int)e.KeyChar <= 57 || e.KeyChar == '.') e.Handled = false;
            else e.Handled = true;

            return;
        }
        private void RangeChange(object Sender, System.EventArgs e)
        {
            int tmpmin = choose_min.Value ,tmpmax = choose_max.Value;
            choose_min.Maximum = tmpmax - 1;
            choose_max.Minimum = tmpmin + 1;
            showrange.Text = "Range:" + tmpmin.ToString() + "~" + tmpmax.ToString();

            return;
        }
        private void Yes_range_Click(object Sender, System.EventArgs e)
        {
            try //防止輸入不合理
            {
                this.gamma = Convert.ToDouble(Gamma.Text);
            }
            catch
            {
                this.gamma = 0.25;
            }
            this.max = choose_max.Value;
            this.min = choose_min.Value;
            this.range.Text="&Range ("+this.min.ToString()+"&~"+this.max.ToString()+") && &Gamma ("+this.gamma.ToString()+")";
            Choose_range.Close(); //關閉視窗

            return;
        }

        private void new_menu() //menu 生成
        {
            MenuItem file = new MenuItem("&File");
            MenuItem load = new MenuItem("&Load");
            MenuItem save = new MenuItem("&Save");
            this.menu.MenuItems.Add(file); //加入 File 選項
            file.MenuItems.Add(load); //File 下的選項 load
            file.MenuItems.Add(save);

            MenuItem overlap_color = new MenuItem("&Overlap");
            MenuItem color_choose = new MenuItem("&Color");
            this.menu.MenuItems.Add(overlap_color);
            overlap_color.MenuItems.Add(color_choose);
            
            MenuItem filter = new MenuItem("&Filter");
            MenuItem mean = new MenuItem("&Mean");
            MenuItem median = new MenuItem("&Median");
            this.menu.MenuItems.Add(filter);
            filter.MenuItems.Add(mean);
            filter.MenuItems.Add(median);

            MenuItem transform = new MenuItem("&Transformation");
            MenuItem log = new MenuItem("&Log");
            MenuItem power = new MenuItem("&Power-Law (need Gamma)");
            this.menu.MenuItems.Add(transform);
            transform.MenuItems.Add(log);
            transform.MenuItems.Add(power);
            transform.MenuItems.Add(this.range);
            
            load.Click += new System.EventHandler(load_image); //按下 open
            save.Click+=new EventHandler(save_image);
            color_choose.Click += new EventHandler(colormenu);
            mean.Click+=new EventHandler(mean_filter);
            median.Click+=new EventHandler(median_filter);
            log.Click+=new EventHandler(log_T);
            power.Click+=new EventHandler(power_T);
            range.Click+=new EventHandler(range_Click);

            return;
        }
        private void load_image(object sender, System.EventArgs e) //按下 menu 中 open
        {
            OpenFileDialog openfile = new OpenFileDialog();
            openfile.Title = "Load";
            openfile.InitialDirectory = "C:\\"; //起始位址
            openfile.Filter = "All(*.*)|*.*|JPeg Image|*.jpg|Bitmap Image|*.bmp|Png Image|*.png"; //開檔類型
            openfile.FilterIndex = 1; //預設顯示的開檔類型
            openfile.RestoreDirectory = true; //存下剛開檔位址

            if (openfile.ShowDialog() == DialogResult.OK)
            {
                try
                {
                    FileStream fs = File.OpenRead(openfile.FileName); //開檔
                    this.file_len = (int)fs.Length; //檔案長度
                    this.src = new Byte[this.file_len]; //new 檔案空間

                    fs.Read(this.src, 0, file_len); //讀檔

                    img1 = byte_to_image(src);
                    img2 = new Bitmap(img1);
                    edge = new Bitmap(img1);
                    this.box1.Image = img1;
                    this.box2.Image = img2;

                    fs.Close(); //關閉檔案
                }
                catch (Exception ex)
                {
                    MessageBox.Show("Error : Could not open file.\nOriginal Error : " + ex.Message);
                }
            }

            return;
        }
        private static Bitmap byte_to_image(byte[] data)
        {
            MemoryStream memory = new MemoryStream();
            byte[] tmp_data = data;
            memory.Write(tmp_data, 0, Convert.ToInt32(tmp_data.Length));
            Bitmap bmp = new Bitmap(memory, false);
            memory.Dispose();

            return bmp;
        }
        private void save_image(object sender, System.EventArgs e)
        {
            SaveFileDialog savefile = new SaveFileDialog();
            savefile.Title = "Save";
            savefile.Filter="JPeg Image|*.jpg|Bitmap Image|*.bmp|Png Image|*.png"; //存檔類型

            if (savefile.ShowDialog() == DialogResult.OK)
            {
                string filename = savefile.FileName.ToString();
                if (filename != "" && filename != null) //有檔名
                {
                    string extname = filename.Substring(filename.LastIndexOf(".") + 1).ToString(); //副檔名
                    System.Drawing.Imaging.ImageFormat imgformat = System.Drawing.Imaging.ImageFormat.Jpeg; //預設 jpg

                    if (extname != "")
                    {
                        switch (extname)
                        {
                            case "jpg":
                                imgformat = System.Drawing.Imaging.ImageFormat.Jpeg;
                                break;
                            case "bmp":
                                imgformat = System.Drawing.Imaging.ImageFormat.Bmp;
                                break;
                            case "png":
                                imgformat = System.Drawing.Imaging.ImageFormat.Png;
                                break;
                            default:
                                MessageBox.Show("只能存取為: jpg,bmp,png 格式");
                                break;
                        }
                    }

                    try
                    {
                        this.box2.Image.Save(filename, imgformat);
                    }
                    catch
                    {
                        MessageBox.Show("Save failed!!");
                    }
                }
            }
        }
        private void range_Click(object sender, System.EventArgs e) //選擇 transform 範圍和 gamma
        {
            choose_min.Value = min;
            choose_max.Value = max;
            Gamma.Text = gamma.ToString();
            Choose_range.ShowDialog();

            return;
        }
        private void colormenu(object sender, System.EventArgs e)
        {
            ColorDialog cd=new ColorDialog();
            if (cd.ShowDialog() == DialogResult.OK) this.color = cd.Color;

            return;
        }
        private void mean_filter(object sender, System.EventArgs e)
        {
            Bitmap img3 = new Bitmap(this.box2.Image);
            BitmapData data = img3.LockBits(new Rectangle(0, 0, img3.Size.Width, img3.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr ptr = data.Scan0;

            BitmapData tmp = img2.LockBits(new Rectangle(0, 0, img2.Size.Width, img2.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr tmpptr = tmp.Scan0;

            unsafe
            {
                int ans = 0, lu = 0, l = 0, ld = 0, mu = 0, m = 0, md = 0, ru = 0, r = 0, rd = 0; //答案 左上 左 左下 中上 中 中下 右上 右 右下
                byte* data_p = (byte*)ptr.ToPointer();
                byte* data_tmp = (byte*)tmpptr.ToPointer();

                for (int i = 0; i < this.img2.Height; ++i)
                    for (int j = 0; j < this.img2.Width; ++j)
                    {
                        if (i == 0 || i == this.img2.Height - 1 || j == 0 || j == this.img2.Width - 1) ans = 0;
                        else
                        {
                            lu = data_tmp[((i - 1) * this.img2.Width + j - 1) * 4];
                            l = data_tmp[(i * this.img2.Width + j - 1) * 4];
                            ld = data_tmp[((i + 1) * this.img2.Width + j - 1) * 4];
                            mu = data_tmp[((i - 1) * this.img2.Width + j) * 4];
                            m = data_tmp[(i * this.img2.Width + j) * 4];
                            md = data_tmp[((i + 1) * this.img2.Width + j) * 4];
                            ru = data_tmp[((i - 1) * this.img2.Width + j + 1) * 4];
                            r = data_tmp[(i * this.img2.Width + j + 1) * 4];
                            rd = data_tmp[((i + 1) * this.img2.Width + j + 1) * 4];
                            ans = (int)((double)(lu + l + ld + mu + m + md + ru + r + rd)/9);
                        }
                        if (ans > 255) ans = 255;
                        if (ans < 0) ans = 0;

                        if (this.horizontal.Checked)
                        {
                            int org = data_p[(i * this.img2.Width + j) * 4];
                            ans = ans * ans + org * org;
                            ans = (int)Math.Sqrt(ans);
                        }

                        data_p[(i * this.img2.Width + j) * 4] = (byte)ans; //B
                        data_p[(i * this.img2.Width + j) * 4 + 1] = (byte)ans; //G
                        data_p[(i * this.img2.Width + j) * 4 + 2] = (byte)ans; //R
                        data_p[(i * this.img2.Width + j) * 4 + 3] = (byte)255; //A
                    }
            }
            img2.UnlockBits(tmp);
            img3.UnlockBits(data);

            this.box2.Image = img3;
            this.box2.Refresh();

            return;
        }
        private void median_filter(object sender, System.EventArgs e)
        {
            Bitmap img3 = new Bitmap(this.box2.Image);
            BitmapData data = img3.LockBits(new Rectangle(0, 0, img3.Size.Width, img3.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr ptr = data.Scan0;

            BitmapData tmp = img2.LockBits(new Rectangle(0, 0, img2.Size.Width, img2.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr tmpptr = tmp.Scan0;

            unsafe
            {
                int ans = 0, lu = 0, l = 0, ld = 0, mu = 0, m = 0, md = 0, ru = 0, r = 0, rd = 0; //答案 左上 左 左下 中上 中 中下 右上 右 右下
                byte* data_p = (byte*)ptr.ToPointer();
                byte* data_tmp = (byte*)tmpptr.ToPointer();

                for (int i = 0; i < this.img2.Height; ++i)
                    for (int j = 0; j < this.img2.Width; ++j)
                    {
                        if (i == 0 || i == this.img2.Height - 1 || j == 0 || j == this.img2.Width - 1) ans = 0;
                        else
                        {
                            lu = data_tmp[((i - 1) * this.img2.Width + j - 1) * 4];
                            l = data_tmp[(i * this.img2.Width + j - 1) * 4];
                            ld = data_tmp[((i + 1) * this.img2.Width + j - 1) * 4];
                            mu = data_tmp[((i - 1) * this.img2.Width + j) * 4];
                            m = data_tmp[(i * this.img2.Width + j) * 4];
                            md = data_tmp[((i + 1) * this.img2.Width + j) * 4];
                            ru = data_tmp[((i - 1) * this.img2.Width + j + 1) * 4];
                            r = data_tmp[(i * this.img2.Width + j + 1) * 4];
                            rd = data_tmp[((i + 1) * this.img2.Width + j + 1) * 4];

                            List<int> find_median = new List<int>();
                            find_median.Add(lu);
                            find_median.Add(l);
                            find_median.Add(ld);
                            find_median.Add(mu);
                            find_median.Add(m);
                            find_median.Add(md);
                            find_median.Add(ru);
                            find_median.Add(r);
                            find_median.Add(rd);
                            find_median.Sort(); //排序

                            ans = find_median[4];
                        }
                        if (ans > 255) ans = 255;
                        if (ans < 0) ans = 0;

                        if (this.horizontal.Checked)
                        {
                            int org = data_p[(i * this.img2.Width + j) * 4];
                            ans = ans * ans + org * org;
                            ans = (int)Math.Sqrt(ans);
                        }

                        data_p[(i * this.img2.Width + j) * 4] = (byte)ans; //B
                        data_p[(i * this.img2.Width + j) * 4 + 1] = (byte)ans; //G
                        data_p[(i * this.img2.Width + j) * 4 + 2] = (byte)ans; //R
                        data_p[(i * this.img2.Width + j) * 4 + 3] = (byte)255; //A
                    }
            }
            img2.UnlockBits(tmp);
            img3.UnlockBits(data);

            this.box2.Image = img3;
            this.box2.Refresh();

            return;
        }
        private void log_T(object sender, System.EventArgs e) //log transform
        {
            Bitmap img3 = new Bitmap(this.box2.Image);
            BitmapData data = img3.LockBits(new Rectangle(0, 0, img3.Size.Width, img3.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr ptr = data.Scan0;

            BitmapData tmp = img2.LockBits(new Rectangle(0, 0, img2.Size.Width, img2.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr tmpptr = tmp.Scan0;

            unsafe
            {
                int ans = 0;
                byte* data_p = (byte*)ptr.ToPointer();
                byte* data_tmp = (byte*)tmpptr.ToPointer();

                for (int i = 0; i < this.img2.Height; ++i)
                    for (int j = 0; j < this.img2.Width; ++j)
                    {
                        ans = (data_p[(i * this.img2.Width + j) * 4] + data_p[(i * this.img2.Width + j) * 4 + 1] + data_p[(i * this.img2.Width + j) * 4 + 2])/3;

                        if (ans >= this.min && ans <= this.max)
                        {
                            double c=this.max-this.min;
                            double r = (ans - this.min) / c;
                            ans = (int)(c * Math.Log(1 + r, 2)) + this.min; //log 2為底 (能還原回原範圍區間)
                        }
                        if (ans > 255) ans = 255;
                        if (ans < 0) ans = 0;

                        data_p[(i * this.img2.Width + j) * 4] = (byte)ans; //B
                        data_p[(i * this.img2.Width + j) * 4 + 1] = (byte)ans; //G
                        data_p[(i * this.img2.Width + j) * 4 + 2] = (byte)ans; //R
                        data_p[(i * this.img2.Width + j) * 4 + 3] = (byte)255; //A
                    }
            }
            img2.UnlockBits(tmp);
            img3.UnlockBits(data);

            this.box2.Image = img3;
            this.box2.Refresh();

            return;
        }
        private void power_T(object sender, System.EventArgs e) //power law transform
        {
            Bitmap img3 = new Bitmap(this.box2.Image);
            BitmapData data = img3.LockBits(new Rectangle(0, 0, img3.Size.Width, img3.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr ptr = data.Scan0;

            BitmapData tmp = img2.LockBits(new Rectangle(0, 0, img2.Size.Width, img2.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr tmpptr = tmp.Scan0;

            unsafe
            {
                int ans = 0;
                byte* data_p = (byte*)ptr.ToPointer();
                byte* data_tmp = (byte*)tmpptr.ToPointer();

                for (int i = 0; i < this.img2.Height; ++i)
                    for (int j = 0; j < this.img2.Width; ++j)
                    {
                        ans = (data_p[(i * this.img2.Width + j) * 4] + data_p[(i * this.img2.Width + j) * 4 + 1] + data_p[(i * this.img2.Width + j) * 4 + 2]) / 3;

                        if (ans >= this.min && ans <= this.max)
                        {
                            double a2b = this.max - this.min;
                            double c = 1;
                            double tmpans = ((double)(ans - this.min)) / a2b;
                            ans = (int)(c * a2b * Math.Pow(tmpans, gamma)) + this.min;
                        }
                        if (ans > 255) ans = 255;
                        if (ans < 0) ans = 0;

                        data_p[(i * this.img2.Width + j) * 4] = (byte)ans; //B
                        data_p[(i * this.img2.Width + j) * 4 + 1] = (byte)ans; //G
                        data_p[(i * this.img2.Width + j) * 4 + 2] = (byte)ans; //R
                        data_p[(i * this.img2.Width + j) * 4 + 3] = (byte)255; //A
                    }
            }
            img2.UnlockBits(tmp);
            img3.UnlockBits(data);

            this.box2.Image = img3;
            this.box2.Refresh();

            return;
        }

        private void new_box(int x ,int y) //生成 picture box
        {
            box1 = new PictureBox();
            box2 = new PictureBox();

            box1.Size = new Size(x, y);
            box2.Size = new Size(x ,y);

            box1.Location = new Point(0, 0);
            box2.Location = new Point(0, 0);

            box1.BackColor = Color.Black;
            box2.BackColor = Color.Black;

            img1 = new Bitmap(HW1.Properties.Resources.default_image);
            img2 = new Bitmap(img1);
            box1.Image = img1;
            box2.Image = img2;

            box1.SizeMode = PictureBoxSizeMode.AutoSize; //自動縮放大小
            box2.SizeMode = PictureBoxSizeMode.AutoSize;

            box1.MouseDown+=new MouseEventHandler(box_MouseDown);
            box1.MouseMove+=new MouseEventHandler(box_MouseMove);
            box2.MouseDown+=new MouseEventHandler(box_MouseDown);
            box2.MouseMove+=new MouseEventHandler(box_MouseMove);

            return;
        }

        private void new_threshold(int x) //(視窗寬)
        {
            this.threshold = new CheckBox();
            this.threshold.Size = new System.Drawing.Size(120, 20);
            this.threshold.Location = new System.Drawing.Point(x - (this.threshold.Size.Width + 10), this.button[2].Location.Y);
            this.threshold.Text = "Thresholding : 127";
            this.threshold.Checked = false;
            this.threshold.Click += new EventHandler(Thresholding);

            this.bar.Size = new System.Drawing.Size(120, 20);
            this.bar.Location = new System.Drawing.Point(x - (this.bar.Size.Width + 20), this.threshold.Location.Y+20);
            this.bar.Maximum = 255;
            this.bar.Minimum = 0;
            this.bar.Value = 127;
            this.bar.TickStyle = TickStyle.BottomRight; //刻度位於下方
            this.bar.TickFrequency = 5; //刻度間隔
            this.bar.LargeChange = 10; //按bar的兩側 移動刻度
            this.bar.SmallChange = 1; //用鍵盤方向鍵 移動刻度
            this.bar.ValueChanged += new EventHandler(Thresholding);
        }
        private void Thresholding(object Sender, System.EventArgs e)
        {
            this.threshold.Text = "Thresholding : " + Convert.ToString(this.bar.Value);

            if (this.threshold.Checked)
            {
                Bitmap img3 = new Bitmap(img2);
                BitmapData data = img3.LockBits(new Rectangle(0, 0, img2.Size.Width, img2.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
                IntPtr ptr = data.Scan0;
                
                unsafe
                {
                    int ans = 0;
                    byte* data_p = (byte*)ptr.ToPointer();

                    for (int i = 0; i < img3.Height; ++i)
                        for (int j = 0; j < img3.Width; ++j)
                        {
                            ans = (data_p[(i * img3.Width + j) * 4] + data_p[(i * img3.Width + j) * 4 + 1] + data_p[(i * img3.Width + j) * 4 + 2]) / 3;

                            if (ans > this.bar.Value) ans = 255;
                            else ans = 0;

                            data_p[(i * img3.Width + j) * 4] = (byte)ans; //B
                            data_p[(i * img3.Width + j) * 4 + 1] = (byte)ans; //G
                            data_p[(i * img3.Width + j) * 4 + 2] = (byte)ans; //R
                            data_p[(i * img3.Width + j) * 4 + 3] = (byte)255; //A
                        }
                }
                img3.UnlockBits(data);

                this.box2.Image = img3;
            }
            else this.box2.Image = img2;
            this.box2.Refresh();

            this.edge = new Bitmap(this.box2.Image);
            this.vertical.Checked = false;
            this.horizontal.Checked = false;

            return;
        }

        private void box_MouseDown(object Sender, MouseEventArgs e)
        {
            if (e.Button == MouseButtons.Left)
            {
                preX = e.X;
                preY = e.Y;
            }

            return;
        }

        private void box_MouseMove(object Sender, MouseEventArgs e)
        {
            if (e.Button==MouseButtons.Left)
            {
                PictureBox box = (PictureBox)Sender;
                box.Refresh();

                int x = e.X - preX + box.Location.X;
                int y = e.Y - preY + box.Location.Y;

                if (x > 0) x = 0;

                if (this.panel1.Size.Width < box.Size.Width) //圖片較寬
                {
                    if (x < this.panel1.Size.Width - box.Size.Width) x = this.panel1.Size.Width - box.Size.Width;
                }
                else x = 0;

                if (y > 0) y = 0;
                if (this.panel1.Size.Height < box.Size.Height) //圖片較長
                {
                    if (y < this.panel1.Size.Height - box.Size.Height) y = this.panel1.Size.Height - box.Size.Height;
                }
                else y = 0;

                box.Location = new Point(x, y);
            }
            preX = e.X;
            preY = e.Y;

            return;
        }

        private void new_panel(int x,int y)
        {
            this.panel1 = new Panel();
            this.panel2 = new Panel();

            panel1.BackColor = Color.Black;
            panel2.BackColor = Color.Black;

            panel1.Size = new Size(x, y);
            panel2.Size = new Size(x, y);

            panel1.AutoScroll = true;
            panel2.AutoScroll = true;

            panel1.Location = new Point(0, 10);
            panel2.Location = new Point(x + 10, 10);

            panel1.Controls.Add(this.box1);
            panel2.Controls.Add(this.box2);

            return;
        }

        private void new_group(int x) //(視窗寬)
        {
            this.R = new RadioButton();
            this.G = new RadioButton();
            this.B = new RadioButton();

            this.group.Size = new System.Drawing.Size(120, 80);
            this.group.Location = new System.Drawing.Point(x - (this.group.Size.Width + 20), 10);

            R.Text = "R";
            G.Text = "G";
            B.Text = "B";

            R.Size = new Size(30, 20);
            G.Size = new Size(30, 20);
            B.Size = new Size(30, 20);

            R.Location = new System.Drawing.Point(20 ,10);//(x - 4 * R.Size.Width, 10);
            G.Location = new System.Drawing.Point(50, 10);
            B.Location = new System.Drawing.Point(80, 10);

            R.Checked = true;
            G.Checked = false;
            B.Checked = false;

            this.group.Controls.Add(this.R);
            this.group.Controls.Add(this.G);
            this.group.Controls.Add(this.B);

            return;
        }

        private void new_button(int len ,int x ,int y) //按鈕生成 (個數 ,視窗寬 ,按鈕間隔)
        {
            int width = 120, height = 40;
            for (int i = 0; i < len; ++i)
            {
                button.Add(new Button());
                this.button[i].Size = new System.Drawing.Size(width, height);
                this.button[i].Location = new System.Drawing.Point(x-(width+20), (height+y) * i+y*2);
                this.button[i].Name = Convert.ToString(i);
            }

            return;
        }

        private void new_edge(int x) //(視窗寬)
        {
            this.vertical = new CheckBox();
            this.horizontal = new CheckBox();

            this.vertical.Size = new Size(120, 20);
            this.horizontal.Size = new Size(120, 20);

            this.vertical.Location = new Point(this.button[4].Location.X+10, this.button[4].Location.Y);
            this.horizontal.Location = new Point(this.button[4].Location.X+10, this.vertical.Location.Y + 20);

            this.vertical.Text = "Vertical";
            this.horizontal.Text = "Horizontal";

            this.vertical.Checked = false;
            this.horizontal.Checked = false;

            this.vertical.Click+=new EventHandler(vertical_sobel);
            this.horizontal.Click+=new EventHandler(horizontal_sobel);

            return;
        }

        private void rgb_extraction(object sender, System.EventArgs e) //按下 RGB extraction
        {
            BitmapData data = img2.LockBits(new Rectangle(0, 0, img2.Size.Width, img2.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr ptr = data.Scan0;
            unsafe
            {
                int tmp = 2, ans = 0;
                byte* data_p = (byte*)ptr.ToPointer();

                if (R.Checked) tmp = 2;
                if (G.Checked) tmp = 1;
                if (B.Checked) tmp = 0;

                for (int i = 0; i < this.img2.Height; ++i)
                    for (int j = 0; j < this.img2.Width; ++j)
                    {
                        ans = data_p[(i * this.img2.Width + j) * 4 + tmp];

                        data_p[(i * this.img2.Width + j) * 4] = (byte)ans; //B
                        data_p[(i * this.img2.Width + j) * 4 + 1] = (byte)ans; //G
                        data_p[(i * this.img2.Width + j) * 4 + 2] = (byte)ans; //R
                        data_p[(i * this.img2.Width + j) * 4 + 3] = (byte)255; //A
                    }
            }
            img2.UnlockBits(data);

            this.box2.Image = img2;
            this.box2.Refresh();

            return;
        }

        private void grayscale(object sender, System.EventArgs e) //按下 RGB extraction
        {
            BitmapData data = img2.LockBits(new Rectangle(0, 0, img2.Size.Width, img2.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr ptr = data.Scan0;
            unsafe
            {
                int ans = 0;
                byte* data_p = (byte*)ptr.ToPointer();

                for (int i = 0; i < this.img2.Height; ++i)
                    for (int j = 0; j < this.img2.Width; ++j)
                    {
                        ans = (data_p[(i * this.img2.Width + j) * 4] + data_p[(i * this.img2.Width + j) * 4 + 1] + data_p[(i * this.img2.Width + j) * 4 + 2])/3;

                        data_p[(i * this.img2.Width + j) * 4] = (byte)ans; //B
                        data_p[(i * this.img2.Width + j) * 4 + 1] = (byte)ans; //G
                        data_p[(i * this.img2.Width + j) * 4 + 2] = (byte)ans; //R
                        data_p[(i * this.img2.Width + j) * 4 + 3] = (byte)255; //A
                    }
            }
            img2.UnlockBits(data);

            this.box2.Image = img2;
            this.box2.Refresh();

            return;
        }

        private void histogram_equalize(object sender, System.EventArgs e)
        {
            BitmapData data = img2.LockBits(new Rectangle(0, 0, img2.Size.Width, img2.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr ptr = data.Scan0;
            unsafe
            {
                int[] cdf = new int[256];
                int cdf_min = 256;
                int ans = 0;
                byte* data_p = (byte*)ptr.ToPointer();

                for (int i = 0; i < this.img2.Height; ++i)
                    for (int j = 0; j < this.img2.Width; ++j)
                    {
                        ans = (data_p[(i * this.img2.Width + j) * 4] + data_p[(i * this.img2.Width + j) * 4 + 1] + data_p[(i * this.img2.Width + j) * 4 + 2]) / 3;
                        
                        if (ans < cdf_min) cdf_min = ans; //最小值
                        ++cdf[ans]; //某值出現次數
                    }
                for (int i = cdf_min + 1; i < 256; ++i) cdf[i] += cdf[i - 1]; //累計次數

                for (int i = 0; i < this.img2.Height; ++i)
                    for (int j = 0; j < this.img2.Width; ++j)
                    {
                        ans = (data_p[(i * this.img2.Width + j) * 4] + data_p[(i * this.img2.Width + j) * 4 + 1] + data_p[(i * this.img2.Width + j) * 4 + 2]) / 3;
                        ans = (int)Math.Round(255.0*(double)(cdf[ans] - cdf[cdf_min]) / (double)(this.img2.Height * this.img2.Width - cdf[cdf_min]) ,1 ,MidpointRounding.AwayFromZero);

                        data_p[(i * this.img2.Width + j) * 4] = (byte)ans; //B
                        data_p[(i * this.img2.Width + j) * 4 + 1] = (byte)ans; //G
                        data_p[(i * this.img2.Width + j) * 4 + 2] = (byte)ans; //R
                        data_p[(i * this.img2.Width + j) * 4 + 3] = (byte)255; //A
                    }
            }
            img2.UnlockBits(data);

            this.box2.Image = img2;
            this.box2.Refresh();

            return;
        }

        private void vertical_sobel(object sender, System.EventArgs e)
        {
            if (this.vertical.Checked)
            {
                Bitmap img3 = new Bitmap(this.box2.Image);
                BitmapData data = img3.LockBits(new Rectangle(0, 0, img3.Size.Width, img3.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
                IntPtr ptr = data.Scan0;

                BitmapData tmp = img2.LockBits(new Rectangle(0, 0, img2.Size.Width, img2.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
                IntPtr tmpptr = tmp.Scan0;

                unsafe
                {
                    int ans = 0, lu = 0, l = 0, ld = 0, ru = 0, r = 0, rd = 0; //答案 左上 左 左下 右上 右 右下
                    byte* data_p = (byte*)ptr.ToPointer();
                    byte* data_tmp = (byte*)tmpptr.ToPointer();

                    for (int i = 0; i < this.img2.Height; ++i)
                        for (int j = 0; j < this.img2.Width; ++j)
                        {
                            if (i == 0 || i == this.img2.Height - 1 || j == 0 || j == this.img2.Width - 1) ans = 0;
                            else
                            {
                                lu = data_tmp[((i - 1) * this.img2.Width + j - 1) * 4];
                                l = data_tmp[(i * this.img2.Width + j - 1) * 4];
                                ld = data_tmp[((i + 1) * this.img2.Width + j - 1) * 4];
                                ru = data_tmp[((i - 1) * this.img2.Width + j + 1) * 4];
                                r = data_tmp[(i * this.img2.Width + j + 1) * 4];
                                rd = data_tmp[((i + 1) * this.img2.Width + j + 1) * 4];
                                ans = -lu - 2 * l - ld + ru + 2 * r + rd;
                            }
                            if (ans > 255) ans = 255;
                            if (ans < 0) ans = 0;

                            if (this.horizontal.Checked)
                            {
                                int org = data_p[(i * this.img2.Width + j) * 4];
                                ans = ans * ans + org * org;
                                ans = (int)Math.Sqrt(ans);
                            }

                            data_p[(i * this.img2.Width + j) * 4] = (byte)ans; //B
                            data_p[(i * this.img2.Width + j) * 4 + 1] = (byte)ans; //G
                            data_p[(i * this.img2.Width + j) * 4 + 2] = (byte)ans; //R
                            data_p[(i * this.img2.Width + j) * 4 + 3] = (byte)255; //A
                        }
                }
                img2.UnlockBits(tmp);
                img3.UnlockBits(data);

                this.box2.Image = img3;
            }
            else
            {
                this.horizontal.Checked = false;
                this.box2.Image = img2;
            }
            this.box2.Refresh();

            return;
        }
        private void horizontal_sobel(object sender, System.EventArgs e)
        {
            if (this.horizontal.Checked)
            {
                Bitmap img3 = new Bitmap(this.box2.Image);
                BitmapData data = img3.LockBits(new Rectangle(0, 0, img3.Size.Width, img3.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
                IntPtr ptr = data.Scan0;

                BitmapData tmp = img2.LockBits(new Rectangle(0, 0, img2.Size.Width, img2.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
                IntPtr tmpptr = tmp.Scan0;

                unsafe
                {
                    int ans = 0, lu = 0, u = 0, ru = 0, ld = 0, d = 0, rd = 0; //答案 左上 上 右下 左下 下 右下
                    byte* data_p = (byte*)ptr.ToPointer();
                    byte* data_tmp = (byte*)tmpptr.ToPointer();

                    for (int i = 0; i < this.img2.Height; ++i)
                        for (int j = 0; j < this.img2.Width; ++j)
                        {
                            if (i == 0 || i == this.img2.Height - 1 || j == 0 || j == this.img2.Width - 1) ans = 0;
                            else
                            {
                                lu = data_tmp[((i - 1) * this.img2.Width + j - 1) * 4];
                                u = data_tmp[((i - 1) * this.img2.Width + j) * 4];
                                ru = data_tmp[((i - 1) * this.img2.Width + j + 1) * 4];
                                ld = data_tmp[((i + 1) * this.img2.Width + j - 1) * 4];
                                d = data_tmp[((i + 1) * this.img2.Width + j) * 4];
                                rd = data_tmp[((i + 1) * this.img2.Width + j + 1) * 4];
                                ans = -lu - 2 * u - ru + ld + 2 * d + rd;
                            }
                            if (ans > 255) ans = 255;
                            if (ans < 0) ans = 0;

                            ans = (int)Math.Sqrt(2 * ans * ans);

                            if (this.vertical.Checked)
                            {
                                int org = data_p[(i * this.img2.Width + j) * 4];
                                ans = ans * ans + org * org;
                                ans = (int)Math.Sqrt(ans);
                            }

                            data_p[(i * this.img2.Width + j) * 4] = (byte)ans; //B
                            data_p[(i * this.img2.Width + j) * 4 + 1] = (byte)ans; //G
                            data_p[(i * this.img2.Width + j) * 4 + 2] = (byte)ans; //R
                            data_p[(i * this.img2.Width + j) * 4 + 3] = (byte)255; //A
                        }
                }
                img2.UnlockBits(tmp);
                img3.UnlockBits(data);

                this.box2.Image = img3;
            }
            else
            {
                this.vertical.Checked = false;
                this.box2.Image = img2;
            }
            this.box2.Refresh();

            return;
        }

        private void overlap(object sender, System.EventArgs e) //覆蓋邊框
        {
            Bitmap detail = new Bitmap(this.box1.Image);
            BitmapData line_data = edge.LockBits(new Rectangle(0, 0, edge.Size.Width, edge.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            BitmapData detail_data = detail.LockBits(new Rectangle(0, 0, detail.Size.Width, detail.Size.Height), ImageLockMode.ReadOnly, PixelFormat.Format32bppArgb);
            IntPtr line_ptr = line_data.Scan0;
            IntPtr detail_ptr = detail_data.Scan0;

            unsafe
            {
                byte* l = (byte*)line_ptr.ToPointer();
                byte* d = (byte*)detail_ptr.ToPointer();

                for (int i = 0; i < detail.Height; ++i)
                    for (int j = 0; j < detail.Width; ++j)
                    {
                        if (l[(i * detail.Width + j) * 4] == 255) //有邊線
                        {
                            d[(i * this.img2.Width + j) * 4] = (byte)this.color.B; //B
                            d[(i * this.img2.Width + j) * 4 + 1] = (byte)this.color.G; //G
                            d[(i * this.img2.Width + j) * 4 + 2] = (byte)this.color.R; //R
                            d[(i * this.img2.Width + j) * 4 + 3] = (byte)255; //A
                        }
                    }
            }
            edge.UnlockBits(line_data);
            detail.UnlockBits(detail_data);
            img2 = new Bitmap(detail);

            this.box2.Image = new Bitmap(img2);
            this.box2.Refresh();

            return;
        }

        private void save_step(object sender, System.EventArgs e) //存下此次操作
        {
            this.img2 = new Bitmap(this.box2.Image);
            this.vertical.Checked = false;
            this.horizontal.Checked = false;
            this.threshold.Checked = false;
            return;
        }
        private void reset(object sender, System.EventArgs e) //reset
        {
            img2 = new Bitmap(img1);
            edge = new Bitmap(img1);
            this.box2.Image = img2;
            this.vertical.Checked = false;
            this.horizontal.Checked = false;
            this.threshold.Checked = false;
            this.bar.Value = 127;
            this.R.Checked = true;
            this.min = 0;
            this.choose_min.Value = 0;
            this.max = 255;
            this.choose_max.Value = 255;
            this.gamma = 0.25;
            this.Gamma.Text = gamma.ToString();
            this.range.Text = "&Range (0~255) && &Gamma (0.25)";

            return;
        }

        /*彈出視窗*/
        private Form Choose_range = new Form();
        private TextBox Gamma = new TextBox();
        private TrackBar choose_min = new TrackBar();
        private TrackBar choose_max = new TrackBar();
        private Label showrange = new Label();
        private Button Yes_range = new Button();

        /*主視窗*/
        private GroupBox group = new GroupBox();
        private RadioButton R, G, B;
        private List<Button> button=new List<Button>();
        private CheckBox threshold, vertical, horizontal;
        private TrackBar bar = new TrackBar();
        private MainMenu menu = new MainMenu();
        private MenuItem range = new MenuItem("&Range (0~255) && &Gamma (0.25)");
        private Byte[] src;
        private Bitmap img1, img2, edge;
        private Color color=new Color();
        private PictureBox box1, box2;
        private Panel panel1 ,panel2;

        /*數值*/
        private int file_len;
        private int preX = 0, preY = 0;
        private int min = 0, max = 255;
        private double gamma = 0.25;
    }
}
