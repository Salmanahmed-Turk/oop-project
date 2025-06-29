//This is a different version with a different class name Bike instead of Motorcycle
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.util.ArrayList;

// Abstract Vehicle class (Abstraction)
abstract class Vehicle {
    protected String plateNumber;
    protected String ownerName;

    public Vehicle(String plateNumber, String ownerName) {
        this.plateNumber = plateNumber;
        this.ownerName = ownerName;
    }

    public String getPlateNumber() { return plateNumber; }
    public String getOwnerName() { return ownerName; }

    public abstract String getType(); // Polymorphism

    public String toDataString() {
        return plateNumber + "," + ownerName + "," + getType();
    }
}

// Car inherits Vehicle
class Car extends Vehicle {
    public Car(String plateNumber, String ownerName) {
        super(plateNumber, ownerName);
    }

    @Override
    public String getType() {
        return "Car";
    }
}

Bike inherits Vehicle
class Bike extends Vehicle {
    public Motorcycle(String plateNumber, String ownerName) {
        super(plateNumber, ownerName);
    }

    @Override
    public String getType() {
        return "Motorcycle";
    }
}

// Parking lot manager with file handling
class ParkingLot {
    private ArrayList<Vehicle> vehicles = new ArrayList<>();
    private final String fileName = "parking_lot.txt";

    public ParkingLot() {
        loadVehicles();
    }

    public void addVehicle(Vehicle v) {
        vehicles.add(v);
        saveVehicles();
    }

    public ArrayList<Vehicle> getVehicles() {
        return vehicles;
    }

    private void saveVehicles() {
        try (BufferedWriter bw = new BufferedWriter(new FileWriter(fileName))) {
            for (Vehicle v : vehicles) {
                bw.write(v.toDataString());
                bw.newLine();
            }
        } catch (IOException e) {
            JOptionPane.showMessageDialog(null, "Error saving vehicles: " + e.getMessage());
        }
    }

    private void loadVehicles() {
        vehicles.clear();
        File file = new File(fileName);
        if (!file.exists()) return;

        try (BufferedReader br = new BufferedReader(new FileReader(fileName))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] parts = line.split(",");
                if (parts.length == 3) {
                    String plate = parts[0];
                    String owner = parts[1];
                    String type = parts[2];
                    if (type.equals("Car")) {
                        vehicles.add(new Car(plate, owner));
                    } else if (type.equals("Motorcycle")) {
                        vehicles.add(new Motorcycle(plate, owner));
                    }
                }
            }
        } catch (IOException e) {
            JOptionPane.showMessageDialog(null, "Error loading vehicles: " + e.getMessage());
        }
    }
}

// Main Dashboard
class Dashboard extends JFrame {
    ParkingLot parkingLot;

    public Dashboard(ParkingLot parkingLot) {
        this.parkingLot = parkingLot;
        setTitle("Parking Lot Management System - Dashboard");
        setSize(400, 250);
        setLocationRelativeTo(null);
        setDefaultCloseOperation(EXIT_ON_CLOSE);

        JButton btnAdd = new JButton("Add Vehicle");
        JButton btnView = new JButton("View Parked Vehicles");

        btnAdd.addActionListener(e -> {
            new AddVehicle(this, parkingLot).setVisible(true);
            this.setVisible(false);
        });

        btnView.addActionListener(e -> {
            new ViewVehicles(this, parkingLot).setVisible(true);
            this.setVisible(false);
        });

        JPanel panel = new JPanel();
        panel.add(btnAdd);
        panel.add(btnView);

        add(panel);
    }
}

// Add Vehicle Window
class AddVehicle extends JFrame {
    Dashboard dashboard;
    ParkingLot parkingLot;

    public AddVehicle(Dashboard dashboard, ParkingLot parkingLot) {
        this.dashboard = dashboard;
        this.parkingLot = parkingLot;
        setTitle("Add Vehicle");
        setSize(350, 250);
        setLocationRelativeTo(null);
        setDefaultCloseOperation(EXIT_ON_CLOSE);

        JLabel lblPlate = new JLabel("Plate Number:");
        JTextField txtPlate = new JTextField(15);

        JLabel lblOwner = new JLabel("Owner Name:");
        JTextField txtOwner = new JTextField(15);

        JLabel lblType = new JLabel("Vehicle Type:");
        JComboBox<String> cbType = new JComboBox<>(new String[]{"Car", "Motorcycle"});

        JButton btnSave = new JButton("Park Vehicle");
        JButton btnBack = new JButton("Back");

        btnSave.addActionListener(e -> {
            try {
                String plate = txtPlate.getText().trim();
                String owner = txtOwner.getText().trim();
                String type = (String) cbType.getSelectedItem();

                if (plate.isEmpty() || owner.isEmpty()) {
                    throw new Exception("Plate Number and Owner Name cannot be empty.");
                }

                Vehicle v;
                if (type.equals("Car")) {
                    v = new Car(plate, owner);
                } else {
                    v = new Motorcycle(plate, owner);
                }

                parkingLot.addVehicle(v);
                JOptionPane.showMessageDialog(this, "Vehicle parked successfully!");
                txtPlate.setText("");
                txtOwner.setText("");
            } catch (Exception ex) {
                JOptionPane.showMessageDialog(this, ex.getMessage());
            }
        });

        btnBack.addActionListener(e -> {
            this.setVisible(false);
            dashboard.setVisible(true);
        });

        setLayout(new GridLayout(5, 2, 5, 5));
        add(lblPlate);
        add(txtPlate);
        add(lblOwner);
        add(txtOwner);
        add(lblType);
        add(cbType);
        add(btnSave);
        add(btnBack);
    }
}

// View Vehicles Window
class ViewVehicles extends JFrame {
    Dashboard dashboard;
    ParkingLot parkingLot;

    public ViewVehicles(Dashboard dashboard, ParkingLot parkingLot) {
        this.dashboard = dashboard;
        this.parkingLot = parkingLot;

        setTitle("Parked Vehicles");
        setSize(500, 350);
        setLocationRelativeTo(null);
        setDefaultCloseOperation(EXIT_ON_CLOSE);

        String[] columns = {"Plate Number", "Owner Name", "Vehicle Type"};
        DefaultTableModel tableModel = new DefaultTableModel(columns, 0);
        JTable table = new JTable(tableModel);

        for (Vehicle v : parkingLot.getVehicles()) {
            Object[] row = {v.getPlateNumber(), v.getOwnerName(), v.getType()};
            tableModel.addRow(row);
        }

        JScrollPane scrollPane = new JScrollPane(table);
        JButton btnBack = new JButton("Back");

        btnBack.addActionListener(e -> {
            this.setVisible(false);
            dashboard.setVisible(true);
        });

        add(scrollPane, BorderLayout.CENTER);
        add(btnBack, BorderLayout.SOUTH);
    }
}

// Main class to run the system
public class ParkingLotManagementSystem {
    public static void main(String[] args) {
        ParkingLot parkingLot = new ParkingLot();
        Dashboard dashboard = new Dashboard(parkingLot);
        dashboard.setVisible(true);
    }
}
