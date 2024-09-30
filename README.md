import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.sql.*;
import java.util.ArrayList;

class AddressEntry {
    private String name;
    private String address;
    private String phoneNumber;

    public AddressEntry(String name, String address, String phoneNumber) {
        this.name = name;
        this.address = address;
        this.phoneNumber = phoneNumber;
    }

    public String getName() {
        return name;
    }

    public String getAddress() {
        return address;
    }

    public String getPhoneNumber() {
        return phoneNumber;
    }
}

public class AddressBookApp extends JFrame {
    private JTextField nameField, addressField, phoneField;
    private JTextArea displayArea;
    private ArrayList<AddressEntry> addressBook;
    private Connection connection;

    public AddressBookApp() {
        super("Address Book");

        try {
            Class.forName("com.mysql.jdbc.Driver");
            connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/addressbook", "root", "root");
            System.out.println("Established");
            Statement stmt = connection.createStatement();
            stmt.execute(
                    "CREATE TABLE IF NOT EXISTS person (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255), address VARCHAR(255), phoneNumber VARCHAR(255))");
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }

        addressBook = new ArrayList<>();

        JPanel inputPanel = new JPanel(new GridLayout(4, 2));
        inputPanel.add(new JLabel("Name:"));
        nameField = new JTextField();
        nameField.setPreferredSize(new Dimension(150, 30));
        inputPanel.add(nameField);
        inputPanel.add(new JLabel("Address:"));
        addressField = new JTextField();
        addressField.setPreferredSize(new Dimension(150, 30));
        inputPanel.add(addressField);
        inputPanel.add(new JLabel("Phone Number:"));
        phoneField = new JTextField();
        phoneField.setPreferredSize(new Dimension(150, 30));
        inputPanel.add(phoneField);

        JButton addButton = new JButton("Add");
        addButton.setPreferredSize(new Dimension(100, 30));
        addButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                String name = nameField.getText();
                String address = addressField.getText();
                String phoneNumber = phoneField.getText();

                if (!name.isEmpty() && !address.isEmpty() && !phoneNumber.isEmpty()) {
                    AddressEntry entry = new AddressEntry(name, address, phoneNumber);
                    addressBook.add(entry);
                    displayAddressBook();
                    clearFields();
                    storeData(name, address, phoneNumber);
                } else {
                    JOptionPane.showMessageDialog(AddressBookApp.this,
                            "Please fill out all fields.", "Error", JOptionPane.ERROR_MESSAGE);
                }
            }
        });

        inputPanel.add(addButton);

        displayArea = new JTextArea(20, 40);
        displayArea.setEditable(false);
        JScrollPane scrollPane = new JScrollPane(displayArea);
        scrollPane.setPreferredSize(new Dimension(400, 400));

        JPanel mainPanel = new JPanel();
        mainPanel.add(inputPanel);
        mainPanel.add(scrollPane);

        add(mainPanel);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        pack();
        setLocationRelativeTo(null);
    }

    private void storeData(String name, String address, String phoneNumber) {
        try {
            String query = "INSERT INTO person(name, address, phoneNumber) VALUES (?, ?, ?)";
            PreparedStatement pstmt = connection.prepareStatement(query);
            pstmt.setString(1, name);
            pstmt.setString(2, address);
            pstmt.setString(3, phoneNumber);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private void displayAddressBook() {
        displayArea.setText("");
        for (AddressEntry entry : addressBook) {
            displayArea.append("Name: " + entry.getName() + "\n");
            displayArea.append("Address: " + entry.getAddress() + "\n");
            displayArea.append("Phone Number: " + entry.getPhoneNumber() + "\n\n");
        }
    }

    private void clearFields() {
        nameField.setText("");
        addressField.setText("");
        phoneField.setText("");
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            public void run() {
                new AddressBookApp().setVisible(true);
            }
        });
    }
}
