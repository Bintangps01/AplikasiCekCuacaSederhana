
# Aplikasi Cek Cuaca Sederhana

Sebuah aplikasi yang dapat digunakan untuk cek cuaca, yang ditujukan untuk menyelesaikan Tugas PBO ke-6.

#### Source Code PenghitungKataFrame.java
```
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import javax.swing.ImageIcon;
import javax.swing.JFileChooser;
import javax.swing.table.DefaultTableModel;
import org.json.JSONArray;
import org.json.JSONObject;

/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 * Click nbfs://nbhost/SystemFileSystem/Templates/GUIForms/JFrame.java to edit this template
 */

/**
 *
 * @author MyBook Z Series
 */
public class CekCuacaSederhanaFrame extends javax.swing.JFrame {

    // API Key dari OpenWeatherMap
    private static final String API_KEY = "50cc76650cc46d1e81b92adec6e918a4";
    private static final String API_URL = "http://api.openweathermap.org/data/2.5/weather?q=%s&appid=%s&units=metric";
    /**
     * Creates new form CekCuacaSederhanaFrame
     */
    public CekCuacaSederhanaFrame() {
        initComponents();
        buttonCekCuaca.addActionListener(e -> cekCuaca());
        
        // Menambahkan ActionListener pada comboFavorit
        comboFavorit.addActionListener(e -> transferKotaKeField());
        
        // Menambahkan ActionListener pada buttonImport dan buttonExport
        buttonImport.addActionListener(e -> importFromCSV());
        buttonExport.addActionListener(e -> exportToCSV());
    }
    
    // Fungsi export data tabel menjadi file csv
    public void exportToCSV() {
        // Ambil model dari JTable
        DefaultTableModel model = (DefaultTableModel) jTable1.getModel();

        // Pilih lokasi dan nama file untuk menyimpan CSV
        JFileChooser fileChooser = new JFileChooser();
        fileChooser.setDialogTitle("Simpan sebagai CSV");
        fileChooser.setSelectedFile(new java.io.File("cuaca.csv"));

        // Memastikan ekstensi CSV saat memilih file
        fileChooser.setFileFilter(new javax.swing.filechooser.FileNameExtensionFilter("CSV File", "csv"));

        int userSelection = fileChooser.showSaveDialog(this);
        if (userSelection == JFileChooser.APPROVE_OPTION) {
            java.io.File fileToSave = fileChooser.getSelectedFile();
            try (FileWriter writer = new FileWriter(fileToSave)) {
                // Menulis header tabel ke file CSV
                for (int columnIndex = 0; columnIndex < model.getColumnCount(); columnIndex++) {
                    writer.write(model.getColumnName(columnIndex));  // Menulis nama kolom
                    if (columnIndex < model.getColumnCount() - 1) {
                        writer.write(",");  // Pisahkan dengan koma
                    }
                }
                writer.write("\n");  // Pindah ke baris baru setelah header

                // Menulis setiap baris data ke file CSV
                for (int rowIndex = 0; rowIndex < model.getRowCount(); rowIndex++) {
                    for (int columnIndex = 0; columnIndex < model.getColumnCount(); columnIndex++) {
                        writer.write(model.getValueAt(rowIndex, columnIndex).toString());
                        if (columnIndex < model.getColumnCount() - 1) {
                            writer.write(";");  // Pisahkan dengan titik koma
                        }
                    }
                    writer.write("\n");  // Pindah ke baris baru setelah setiap baris data
                }

                // Menampilkan pesan bahwa ekspor berhasil
                jOptionPane1.showMessageDialog(this, "Data berhasil diekspor ke " + fileToSave.getAbsolutePath());
            } catch (IOException e) {
                jOptionPane1.showMessageDialog(this, "Terjadi kesalahan saat menyimpan file: " + e.getMessage());
            }
        }
    }
    
    // Fungsi untuk menyimpan data tabel menjadi file csv
    public void importFromCSV() {
        // Membuka file CSV menggunakan JFileChooser
        JFileChooser fileChooser = new JFileChooser();
        fileChooser.setDialogTitle("Pilih File CSV");
        fileChooser.setFileFilter(new javax.swing.filechooser.FileNameExtensionFilter("CSV File", "csv"));

        int userSelection = fileChooser.showOpenDialog(this);
        if (userSelection == JFileChooser.APPROVE_OPTION) {
            java.io.File fileToOpen = fileChooser.getSelectedFile();

            try (BufferedReader br = new BufferedReader(new FileReader(fileToOpen))) {
                // Membaca baris pertama untuk header tabel
                String line = br.readLine();
                if (line != null) {
                    String[] headers = line.split(",");

                    // Membuat model tabel baru dengan header yang diambil dari CSV
                    DefaultTableModel model = new DefaultTableModel();
                    for (String header : headers) {
                        model.addColumn(header.trim());  // Menambahkan header ke model
                    }

                    // Membaca data tabel dan menambahkannya ke model
                    while ((line = br.readLine()) != null) {
                        String[] data = line.split(";");
                        model.addRow(data);  // Menambahkan baris data ke tabel
                    }
                    
                    // Setelah data berhasil diimpor ke tabel, iterasikan setiap baris
                    for (int i = 0; i < model.getRowCount(); i++) {
                        String namaKota = (String) model.getValueAt(i, 0); // Ambil data dari kolom pertama
                        if (!isCityInCombo(namaKota)) { // Periksa jika kota belum ada di comboKota
                            comboFavorit.addItem(namaKota); // Tambahkan kota ke comboKota
                        }
                    }
                    // Mengatur model baru ke JTable
                    jTable1.setModel(model);
                    jOptionPane1.showMessageDialog(this, "Data berhasil diimpor dari " + fileToOpen.getAbsolutePath());
                } else {
                    jOptionPane1.showMessageDialog(this, "File kosong atau format tidak valid.");
                }
            } catch (IOException e) {
                jOptionPane1.showMessageDialog(this, "Terjadi kesalahan saat membuka file: " + e.getMessage());
            }
        }
    }
    
    // Fungsi untuk mentransfer pilihan ComboBox ke TextField
    private void transferKotaKeField() {
        // Mendapatkan kota yang dipilih dari comboFavorit
        String kotaTerpilih = (String) comboFavorit.getSelectedItem();

        // Memasukkan nama kota yang dipilih ke fieldKota
        if (kotaTerpilih != null) {
            fieldKota.setText(kotaTerpilih);
        }
    }

    // Mapping kode kondisi cuaca OpenWeatherMap ke deskripsi dalam bahasa Indonesia
    private String getKondisiCuacaInIndonesian(String kondisiCuaca) {
        switch (kondisiCuaca) {
            case "clear sky":
                return "Langit Cerah";
            case "few clouds":
                return "Sedikit Berawan";
            case "scattered clouds":
                return "Awan Tersebar";
            case "broken clouds":
                return "Cenderung Berawan";
            case "overcast clouds":
                return "Berawan Tebal";
            case "light rain":
                return "Hujan Ringan";
            case "moderate rain":
                return "Hujan Sedang";
            case "heavy rain":
                return "Hujan Deras";
            case "very heavy rain":
                return "Hujan Sangat Deras";
            case "freezing rain":
                return "Hujan Es";
            case "light intensity drizzle":
                return "Gerimis Ringan";
            case "heavy intensity drizzle":
                return "Gerimis Berat";
            case "thunderstorm with light rain":
                return "Petir dengan Hujan Ringan";
            case "thunderstorm with rain":
                return "Petir dengan Hujan";
            case "thunderstorm with heavy rain":
                return "Petir dengan Hujan Deras";
            case "light snow":
                return "Salju Ringan";
            case "moderate snow":
                return "Salju Sedang";
            case "heavy snow":
                return "Salju Deras";
            case "sleet":
                return "Salju dan Hujan";
            case "mist":
                return "Kabut Ringan";
            case "smoke":
                return "Asap";
            case "haze":
                return "Kabut Asap";
            case "dust":
                return "Debu";
            case "fog":
                return "Kabut Tebal";
            case "sand":
                return "Pasir";
            case "ash":
                return "Abu";
            case "squall":
                return "Badai Angin";
            case "tornado":
                return "Tornado";
            case "extreme":
                return "Cuaca Ekstrem";
            default:
                return "Kondisi Cuaca Tidak Diketahui";
        }
    }

   // Fungsi untuk mengecek cuaca berdasarkan nama kota yang dimasukkan
    private void cekCuaca() {
        String kota = fieldKota.getText().trim();  // Mendapatkan nama kota dari input
        if (kota.isEmpty()) {  // Memastikan kota tidak kosong
            jOptionPane1.showMessageDialog(this, "Harap masukkan nama kota!");  // Jika kosong, tampilkan pesan error
            return;
        }

        try {
            // Membentuk URL untuk request API berdasarkan kota yang dimasukkan
            String urlString = String.format(API_URL, kota, API_KEY);
            URL url = new URL(urlString);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");
            connection.setConnectTimeout(5000);  // Set timeout koneksi
            connection.setReadTimeout(5000);  // Set timeout baca data

            InputStreamReader reader = new InputStreamReader(connection.getInputStream());
            StringBuilder response = new StringBuilder();
            int charRead;
            while ((charRead = reader.read()) != -1) {
                response.append((char) charRead);
            }

            // Parse JSON response dari API
            JSONObject jsonResponse = new JSONObject(response.toString());

            if (jsonResponse.has("weather")) {  // Jika ada data cuaca
                JSONArray weatherArray = jsonResponse.getJSONArray("weather");
                JSONObject weather = weatherArray.getJSONObject(0);
                String kondisiCuaca = weather.getString("description");  // Mendapatkan deskripsi cuaca

                // Mendapatkan data suhu dan kelembaban
                JSONObject main = jsonResponse.getJSONObject("main");
                double temp = main.getDouble("temp");
                double humidity = main.getDouble("humidity");

                // Terjemahkan kondisi cuaca ke bahasa Indonesia
                String kondisiCuacaIndo = getKondisiCuacaInIndonesian(kondisiCuaca);

                // Menampilkan informasi cuaca dalam label
                String cuacaInfo = String.format("Suhu: %.1f°C, Kelembaban: %.1f%%, Kondisi: %s", temp, humidity, kondisiCuacaIndo);
                labelStatusCuaca.setText(cuacaInfo);

                // Menambahkan kota ke comboFavorit jika belum ada
                if (!isCityInCombo(kota)) {
                    comboFavorit.addItem(kota);
                }

                // Cek jika kota sudah ada dalam tabel
                DefaultTableModel model = (DefaultTableModel) jTable1.getModel();
                boolean cityExists = false;
                int rowIndex = -1;

                // Periksa apakah kota sudah ada di tabel
                for (int i = 0; i < model.getRowCount(); i++) {
                    String existingCity = (String) model.getValueAt(i, 0);
                    if (existingCity.equals(kota)) {
                        cityExists = true;
                        rowIndex = i;
                        break;
                    }
                }

                // Jika kota sudah ada, update status cuaca
                if (cityExists) {
                    model.setValueAt(cuacaInfo, rowIndex, 1);  // Update status cuaca untuk kota yang ditemukan
                } else {
                    // Jika kota belum ada, tambahkan baris baru dengan data cuaca
                    model.addRow(new Object[]{kota, cuacaInfo});
                }

                // Menampilkan gambar cuaca sesuai dengan kondisi cuaca pada labelGambarCuaca
                ImageIcon cuacaIcon = getCuacaIcon(kondisiCuaca);
                labelGambarCuaca.setIcon(cuacaIcon);  // Menampilkan gambar cuaca di labelGambarCuaca
            }
        } catch (Exception e) {  // Jika terjadi error saat mengambil atau memproses data
            jOptionPane1.showMessageDialog(this, "Terjadi kesalahan: " + e.getMessage());  // Tampilkan pesan error
        }
    }

    // Fungsi untuk mengecek apakah kota sudah ada di dalam tabel
    private boolean isCityInTable(String kota) {
        DefaultTableModel model = (DefaultTableModel) jTable1.getModel();
        for (int i = 0; i < model.getRowCount(); i++) {
            String existingCity = (String) model.getValueAt(i, 0);  // Mengambil nama kota yang ada dalam tabel
            if (existingCity.equalsIgnoreCase(kota)) {
                return true;  // Kota sudah ada
            }
        }
        return false;  // Kota belum ada
    }
    
    // Cek apakah kota sudah ada di dalam comboFavorit
    private boolean isCityInCombo(String kota) {
        for (int i = 0; i < comboFavorit.getItemCount(); i++) {
            if (comboFavorit.getItemAt(i).equals(kota)) {
                return true;
            }
        }
        return false;
    }
    
    // Mengambil gambar cuaca sesuai kondisi cuaca
    private ImageIcon getCuacaIcon(String kondisiCuaca) {
        // Mendapatkan ID ikon cuaca dari respon API
        String iconId = getCuacaIconId(kondisiCuaca);

        if (iconId == null || iconId.isEmpty()) {
            return new ImageIcon();  // Mengembalikan ImageIcon kosong jika tidak ada ikon
        }

        // URL base untuk ikon cuaca OpenWeatherMap
        String baseUrl = "http://openweathermap.org/img/wn/";

        // Membuat URL lengkap untuk mengambil gambar berdasarkan iconId
        String iconUrl = baseUrl + iconId + "@2x.png";  // Format gambar (2x resolusi)

        try {
            URL url = new URL(iconUrl);
            return new ImageIcon(url);  // Mengembalikan gambar dari URL
        } catch (Exception e) {
            e.printStackTrace();
            return new ImageIcon();  // Mengembalikan ImageIcon kosong jika terjadi error
        }
    }

    // Mendapatkan ID ikon cuaca berdasarkan kondisi
    private String getCuacaIconId(String kondisiCuaca) {
        switch (kondisiCuaca) {
            case "clear sky":
                return "01d";  // Siang cerah
            case "few clouds":
                return "02d";  // Sedikit berawan siang
            case "scattered clouds":
                return "03d";  // Awan tersebar siang
            case "broken clouds":
                return "04d";  // Berawan sebagian siang
            case "overcast clouds":
                return "04d";  // Berawan tebal
            case "light rain":
                return "10d";  // Hujan ringan siang
            case "moderate rain":
                return "10d";  // Hujan sedang siang
            case "heavy rain":
                return "10d";  // Hujan deras siang
            case "very heavy rain":
                return "10d";  // Hujan sangat deras
            case "freezing rain":
                return "13d";  // Hujan es
            case "light intensity drizzle":
                return "09d";  // Gerimis ringan
            case "heavy intensity drizzle":
                return "09d";  // Gerimis berat
            case "thunderstorm with light rain":
                return "11d";  // Petir dengan hujan ringan
            case "thunderstorm with rain":
                return "11d";  // Petir dengan hujan
            case "thunderstorm with heavy rain":
                return "11d";  // Petir dengan hujan deras
            case "light snow":
                return "13d";  // Salju ringan
            case "moderate snow":
                return "13d";  // Salju sedang
            case "heavy snow":
                return "13d";  // Salju deras
            case "sleet":
                return "13d";  // Salju dan hujan
            case "mist":
                return "50d";  // Kabut ringan
            case "smoke":
                return "50d";  // Asap
            case "haze":
                return "50d";  // Kabut asap
            case "dust":
                return "50d";  // Debu
            case "fog":
                return "50d";  // Kabut tebal
            case "sand":
                return "50d";  // Pasir
            case "ash":
                return "50d";  // Abu
            case "squall":
                return "50d";  // Badai angin
            case "tornado":
                return "50d";  // Tornado
            case "extreme":
                return "50d";  // Cuaca ekstrem
            default:
                return "";  // Kembalikan string kosong jika kondisi cuaca tidak dikenali
        }
    }
    /**
     * This method is called from within the constructor to initialize the form.
     * WARNING: Do NOT modify this code. The content of this method is always
     * regenerated by the Form Editor.
     */
    @SuppressWarnings("unchecked")
    // <editor-fold defaultstate="collapsed" desc="Generated Code">                          
    private void initComponents() {
        java.awt.GridBagConstraints gridBagConstraints;

        jOptionPane1 = new javax.swing.JOptionPane();
        panelUtama = new javax.swing.JPanel();
        jLabel1 = new javax.swing.JLabel();
        fieldKota = new javax.swing.JTextField();
        jLabel2 = new javax.swing.JLabel();
        buttonCekCuaca = new javax.swing.JButton();
        comboFavorit = new javax.swing.JComboBox<>();
        panelStatus = new javax.swing.JPanel();
        labelStatusCuaca = new javax.swing.JLabel();
        labelGambarCuaca = new javax.swing.JLabel();
        jPanel1 = new javax.swing.JPanel();
        buttonImport = new javax.swing.JButton();
        buttonExport = new javax.swing.JButton();
        jScrollPane1 = new javax.swing.JScrollPane();
        jTable1 = new javax.swing.JTable();
        jScrollPane2 = new javax.swing.JScrollPane();

        setDefaultCloseOperation(javax.swing.WindowConstants.EXIT_ON_CLOSE);
        setTitle("Cek Cuaca");
        setPreferredSize(new java.awt.Dimension(558, 710));

        panelUtama.setBackground(new java.awt.Color(255, 255, 204));
        panelUtama.setToolTipText("");
        panelUtama.setName(""); // NOI18N
        panelUtama.setPreferredSize(new java.awt.Dimension(558, 510));
        panelUtama.setLayout(new java.awt.GridBagLayout());

        jLabel1.setFont(new java.awt.Font("Segoe UI Variable", 1, 18)); // NOI18N
        jLabel1.setText("Aplikasi Cek Cuaca Sederhana");
        gridBagConstraints = new java.awt.GridBagConstraints();
        gridBagConstraints.gridx = 0;
        gridBagConstraints.gridy = 0;
        gridBagConstraints.insets = new java.awt.Insets(10, 80, 0, 80);
        panelUtama.add(jLabel1, gridBagConstraints);

        fieldKota.addFocusListener(new java.awt.event.FocusAdapter() {
            public void focusGained(java.awt.event.FocusEvent evt) {
                fieldKotaFocusGained(evt);
            }
        });
        gridBagConstraints = new java.awt.GridBagConstraints();
        gridBagConstraints.gridx = 0;
        gridBagConstraints.gridy = 2;
        gridBagConstraints.ipadx = 100;
        gridBagConstraints.insets = new java.awt.Insets(8, 0, 8, 0);
        panelUtama.add(fieldKota, gridBagConstraints);

        jLabel2.setText("Masukan Nama Kota");
        gridBagConstraints = new java.awt.GridBagConstraints();
        gridBagConstraints.gridx = 0;
        gridBagConstraints.gridy = 1;
        gridBagConstraints.insets = new java.awt.Insets(13, 0, 8, 0);
        panelUtama.add(jLabel2, gridBagConstraints);

        buttonCekCuaca.setText("Cek Cuaca");
        gridBagConstraints = new java.awt.GridBagConstraints();
        gridBagConstraints.gridx = 0;
        gridBagConstraints.gridy = 4;
        gridBagConstraints.insets = new java.awt.Insets(8, 0, 8, 0);
        panelUtama.add(buttonCekCuaca, gridBagConstraints);

        gridBagConstraints = new java.awt.GridBagConstraints();
        gridBagConstraints.gridx = 0;
        gridBagConstraints.gridy = 3;
        panelUtama.add(comboFavorit, gridBagConstraints);

        panelStatus.setBackground(new java.awt.Color(255, 255, 204));
        panelStatus.setBorder(javax.swing.BorderFactory.createTitledBorder(null, "Status Cuaca", javax.swing.border.TitledBorder.CENTER, javax.swing.border.TitledBorder.DEFAULT_POSITION, new java.awt.Font("Segoe UI Variable", 1, 18))); // NOI18N
        panelStatus.setLayout(new java.awt.GridBagLayout());

        labelStatusCuaca.setText("Hasil Akan Muncul Disini");
        gridBagConstraints = new java.awt.GridBagConstraints();
        gridBagConstraints.gridx = 0;
        gridBagConstraints.gridy = 1;
        panelStatus.add(labelStatusCuaca, gridBagConstraints);
        panelStatus.add(labelGambarCuaca, new java.awt.GridBagConstraints());

        gridBagConstraints = new java.awt.GridBagConstraints();
        gridBagConstraints.gridx = 0;
        gridBagConstraints.gridy = 5;
        gridBagConstraints.fill = java.awt.GridBagConstraints.BOTH;
        gridBagConstraints.ipadx = 1;
        gridBagConstraints.ipady = 1;
        gridBagConstraints.insets = new java.awt.Insets(6, 18, 20, 18);
        panelUtama.add(panelStatus, gridBagConstraints);

        jPanel1.setBackground(new java.awt.Color(255, 255, 204));

        buttonImport.setText("Import");
        jPanel1.add(buttonImport);

        buttonExport.setText("Export");
        jPanel1.add(buttonExport);

        gridBagConstraints = new java.awt.GridBagConstraints();
        gridBagConstraints.gridx = 0;
        gridBagConstraints.gridy = 6;
        panelUtama.add(jPanel1, gridBagConstraints);

        jTable1.setModel(new javax.swing.table.DefaultTableModel(
            new Object [][] {

            },
            new String [] {
                "Kota", "Status Cuaca"
            }
        ) {
            Class[] types = new Class [] {
                java.lang.String.class, java.lang.String.class
            };

            public Class getColumnClass(int columnIndex) {
                return types [columnIndex];
            }
        });
        jScrollPane1.setViewportView(jTable1);
        if (jTable1.getColumnModel().getColumnCount() > 0) {
            jTable1.getColumnModel().getColumn(0).setMaxWidth(150);
            jTable1.getColumnModel().getColumn(1).setMinWidth(200);
        }

        gridBagConstraints = new java.awt.GridBagConstraints();
        gridBagConstraints.gridx = 0;
        gridBagConstraints.gridy = 7;
        gridBagConstraints.ipadx = 480;
        gridBagConstraints.ipady = 100;
        panelUtama.add(jScrollPane1, gridBagConstraints);

        getContentPane().add(panelUtama, java.awt.BorderLayout.CENTER);
        getContentPane().add(jScrollPane2, java.awt.BorderLayout.PAGE_START);

        pack();
    }// </editor-fold>                        

    private void fieldKotaFocusGained(java.awt.event.FocusEvent evt) {                                      
        // Menghapus teks ketika mendapat fokus
        fieldKota.setText("");
    }                                     

    /**
     * @param args the command line arguments
     */
    public static void main(String args[]) {
        /* Set the Nimbus look and feel */
        //<editor-fold defaultstate="collapsed" desc=" Look and feel setting code (optional) ">
        /* If Nimbus (introduced in Java SE 6) is not available, stay with the default look and feel.
         * For details see http://download.oracle.com/javase/tutorial/uiswing/lookandfeel/plaf.html 
         */
        try {
            for (javax.swing.UIManager.LookAndFeelInfo info : javax.swing.UIManager.getInstalledLookAndFeels()) {
                if ("Nimbus".equals(info.getName())) {
                    javax.swing.UIManager.setLookAndFeel(info.getClassName());
                    break;
                }
            }
        } catch (ClassNotFoundException ex) {
            java.util.logging.Logger.getLogger(CekCuacaSederhanaFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        } catch (InstantiationException ex) {
            java.util.logging.Logger.getLogger(CekCuacaSederhanaFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        } catch (IllegalAccessException ex) {
            java.util.logging.Logger.getLogger(CekCuacaSederhanaFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        } catch (javax.swing.UnsupportedLookAndFeelException ex) {
            java.util.logging.Logger.getLogger(CekCuacaSederhanaFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        }
        //</editor-fold>

        /* Create and display the form */
        java.awt.EventQueue.invokeLater(new Runnable() {
            public void run() {
                new CekCuacaSederhanaFrame().setVisible(true);
            }
        });
    }

    // Variables declaration - do not modify                     
    private javax.swing.JButton buttonCekCuaca;
    private javax.swing.JButton buttonExport;
    private javax.swing.JButton buttonImport;
    private javax.swing.JComboBox<String> comboFavorit;
    private javax.swing.JTextField fieldKota;
    private javax.swing.JLabel jLabel1;
    private javax.swing.JLabel jLabel2;
    private javax.swing.JOptionPane jOptionPane1;
    private javax.swing.JPanel jPanel1;
    private javax.swing.JScrollPane jScrollPane1;
    private javax.swing.JScrollPane jScrollPane2;
    private javax.swing.JTable jTable1;
    private javax.swing.JLabel labelGambarCuaca;
    private javax.swing.JLabel labelStatusCuaca;
    private javax.swing.JPanel panelStatus;
    private javax.swing.JPanel panelUtama;
    // End of variables declaration                   
}
```
## Fitur Utama
- Menggunakan API OpenWeatherMap untuk mendapatkan status cuaca
- penampilan gambar cuaca

## Fitur Tambahan (Variasi)
- Kota tersimpan di ComboBox
- Export data dari tabel
- Import data ke tabel
## Referensi

 - [Modul PBO2 Tugas 6](https://drive.google.com/file/d/103_sn-_6Xi8iW7AdfUe02QrCW83vK1oL/view)


## Biodata Pembuat
- Nama : Bintang Putra Setiawan
- Kelas : 5B Reg Pagi Banjarmasin
- NPM : 2210010539
- [@Bintangps01](https://github.com/Bintangps01)
