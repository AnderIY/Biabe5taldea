package GUI;
import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import javax.swing.table.TableRowSorter;

import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.sql.*;
import java.util.List;
import java.util.ArrayList;

public class Taulak extends JFrame {
    private DefaultTableModel tableModel;
    private JTable table;
    private JComboBox<String> tableSelector;
    private JButton selectButton;
    private JButton insertButton;
    private JButton deleteButton;
    private Connection connection;

    public Taulak() {
    	getContentPane().setBackground(new Color(21, 31, 66));
    	setBackground(new Color(21, 31, 66));
        setTitle("BiabeDB");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(748, 459);

        tableModel = new DefaultTableModel();
        table = new JTable(tableModel);
        table.setForeground(new Color(255, 255, 255));
        table.setBackground(new Color(21, 31, 66));
        JScrollPane scrollPane = new JScrollPane(table);

        tableSelector = new JComboBox<>();
        tableSelector.setPreferredSize(new Dimension(150, 25));

        selectButton = new JButton("Bilatu");
        selectButton.setBackground(new Color(255, 255, 255));
        insertButton = new JButton("Sartu");
        deleteButton = new JButton("Ezabatu");

        JPanel panel = new JPanel();
        panel.setBackground(new Color(255, 255, 255));
        panel.setForeground(new Color(21, 31, 66));
        panel.add(tableSelector);
        panel.add(selectButton);
        panel.add(insertButton);
        panel.add(deleteButton);

        getContentPane().add(scrollPane, BorderLayout.CENTER);
        getContentPane().add(panel, BorderLayout.SOUTH);
        TableRowSorter<DefaultTableModel> sorter = new TableRowSorter<>(tableModel);
        table.setRowSorter(sorter);
        JButton modifyButton = new JButton("Modifikatu");
        modifyButton.setBackground(new Color(255, 255, 255));
        panel.add(modifyButton);

        modifyButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                showUpdateDialog((String) tableSelector.getSelectedItem());
            }
        });

        getContentPane().add(panel, BorderLayout.SOUTH);
        selectButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                showSelectFieldsDialog((String) tableSelector.getSelectedItem());
            }
        });

        insertButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                showInsertDialog((String) tableSelector.getSelectedItem());
            }
        });

        deleteButton.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                String referenceToDelete = JOptionPane.showInputDialog("Sartu ezabatu nahi duzun 'nan' edo 'ID' mesedez:");

                if (referenceToDelete != null && !referenceToDelete.isEmpty()) {
                    String tableName = (String) tableSelector.getSelectedItem();

                    int rowCount = table.getRowCount();
                    int referenceColumnIndex = -1;

                    // Buscar el campo "nan"
                    for (int i = 0; i < table.getColumnCount(); i++) {
                        String columnName = tableModel.getColumnName(i);
                        if (columnName.equals("nan")) {
                            referenceColumnIndex = i;
                            break;
                        }
                    }

                    // Si no se encuentra el campo "nan", buscar el campo "ID"
                    if (referenceColumnIndex == -1) {
                        for (int i = 0; i < table.getColumnCount(); i++) {
                            String columnName = tableModel.getColumnName(i);
                            if (columnName.equals("ID")) {
                                referenceColumnIndex = i;
                                break;
                            }
                        }
                    }

                    if (referenceColumnIndex != -1) {
                        int rowToDelete = -1;

                        for (int i = 0; i < rowCount; i++) {
                            String value = tableModel.getValueAt(i, referenceColumnIndex).toString();
                            if (value.equals(referenceToDelete)) {
                                rowToDelete = i;
                                break;
                            }
                        }

                        if (rowToDelete != -1) {
                            int confirm = JOptionPane.showConfirmDialog(null, "Seguro erregistro:'" + tableModel.getColumnName(referenceColumnIndex) + "'ezabatu nahi duzula? " + referenceToDelete + "?", "Confirmación", JOptionPane.YES_NO_OPTION);

                            if (confirm == JOptionPane.YES_OPTION) {
                                deleteDataFromDatabase(tableName, tableModel.getColumnName(referenceColumnIndex), referenceToDelete);
                                tableModel.removeRow(rowToDelete);
                            }
                        } else {
                            JOptionPane.showMessageDialog(null, "Ez da aurkitu'" + tableModel.getColumnName(referenceColumnIndex) + "'.", "Aviso", JOptionPane.INFORMATION_MESSAGE);
                        }
                    } else {
                        JOptionPane.showMessageDialog(null, "Erreferentzia ez da aurkitu ('nan' o 'ID').", "Error", JOptionPane.ERROR_MESSAGE);
                    }
                } else {
                    JOptionPane.showMessageDialog(null, "Sartu balore egokia", "Error", JOptionPane.ERROR_MESSAGE);
                }
            }
        });
        tableSelector.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                String selectedTable = (String) tableSelector.getSelectedItem();
                selectDataFromDatabase(selectedTable);
            }
        });

        try {
            String url = "jdbc:mysql://192.168.5.1:3306/biabe5taldea";
            String usuario = "root";
            String contrasena = "";
            connection = DriverManager.getConnection(url, usuario, contrasena);

           
            List<String> tableNames = getTableNames(connection);
            for (String tableName : tableNames) {
                tableSelector.addItem(tableName);
            }
        } catch (SQLException e) {
            e.printStackTrace();
            JOptionPane.showMessageDialog(null, "Error konektatzean", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    public void habilitarEdicion() {
        table.setEnabled(true); // Habilitar edición en la tabla
    }

    public void deshabilitarEdicion() {
        table.setEnabled(false); // Deshabilitar edición en la tabla
    }

    private List<String> getTableNames(Connection connection) {
        List<String> tableNames = new ArrayList<>();
        try {
            DatabaseMetaData metaData = connection.getMetaData();
            ResultSet resultSet = metaData.getTables(null, null, null, new String[]{"TABLE"});
            while (resultSet.next()) {
                String tableName = resultSet.getString("TABLE_NAME");
                if (!tableName.startsWith("pma_") && !tableName.equals("usuarios")) {
                    tableNames.add(tableName);
                }
            }
            resultSet.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return tableNames;
    }
    private String findReferenceColumn() {
        String tableName = (String) tableSelector.getSelectedItem();
        List<String> columnNames = getColumnNames(tableName);

        if (columnNames.contains("nan")) {
            return "nan";
        } else if (columnNames.contains("ID")) {
            return "ID";
        } else {
            return null;
        }
    }
    private void showDeleteDialog(String referenceColumn) {
        String referenceToDelete = JOptionPane.showInputDialog("Sartu ezabatu nahi duzun balorea '" + referenceColumn);

        if (referenceToDelete != null && !referenceToDelete.isEmpty()) {
            String tableName = (String) tableSelector.getSelectedItem();

            int rowCount = table.getRowCount();
            int referenceColumnIndex = -1;

            for (int i = 0; i < table.getColumnCount(); i++) {
                String columnName = tableModel.getColumnName(i);
                if (columnName.equals(referenceColumn)) {
                    referenceColumnIndex = i;
                    break;
                }
            }

            if (referenceColumnIndex != -1) {
                int rowToDelete = -1;

                for (int i = 0; i < rowCount; i++) {
                    String value = tableModel.getValueAt(i, referenceColumnIndex).toString();
                    if (value.equals(referenceToDelete)) {
                        rowToDelete = i;
                        break;
                    }
                }

                if (rowToDelete != -1) {
                    int confirm = JOptionPane.showConfirmDialog(null, "¿Seguro que deseas eliminar el registro con '" + referenceColumn + "': " + referenceToDelete + "?", "Confirmación", JOptionPane.YES_NO_OPTION);

                    if (confirm == JOptionPane.YES_OPTION) {
                        deleteDataFromDatabase(tableName, referenceColumn, referenceToDelete);
                        tableModel.removeRow(rowToDelete);
                    }
                } else {
                    JOptionPane.showMessageDialog(null, "No se encontró el valor en el campo '" + referenceColumn + "'.", "Aviso", JOptionPane.INFORMATION_MESSAGE);
                }
            } else {
                JOptionPane.showMessageDialog(null, "No se encontró el campo de referencia.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        } else {
            JOptionPane.showMessageDialog(null, "Introduce un valor válido para el campo de referencia.", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }
    private void showSelectFieldsDialog(String tableName) {
        List<String> allFields = getColumnNames(tableName);

        // Crear un array de checkboxes para cada campo
        JCheckBox[] fieldCheckboxes = new JCheckBox[allFields.size()];
        for (int i = 0; i < allFields.size(); i++) {
            fieldCheckboxes[i] = new JCheckBox(allFields.get(i));
            fieldCheckboxes[i].setSelected(true); // Por defecto, seleccionar todos
        }

        // Crear un panel con checkboxes
        JPanel panel = new JPanel(new GridLayout(0, 1));
        for (JCheckBox checkbox : fieldCheckboxes) {
            panel.add(checkbox);
        }

        // Agregar campo de búsqueda
        JTextField searchField = new JTextField(20);
        panel.add(new JLabel("Buscar:"));
        panel.add(searchField);

        // Mostrar el diálogo de selección
        int result = JOptionPane.showConfirmDialog(null, panel, "Seleccionar Campos", JOptionPane.OK_CANCEL_OPTION);

        if (result == JOptionPane.OK_OPTION) {
            // Obtener los campos seleccionados
            List<String> selectedFields = new ArrayList<>();
            for (JCheckBox checkbox : fieldCheckboxes) {
                if (checkbox.isSelected()) {
                    selectedFields.add(checkbox.getText());
                }
            }

            // Obtener la palabra clave de búsqueda
            String searchKeyword = searchField.getText();

            // Realizar la consulta y mostrar los datos
            selectDataWithFieldsAndSearch(tableName, selectedFields, searchKeyword);
        }
    }

    
    private void selectDataWithFieldsAndSearch(String tableName, List<String> selectedFields, String searchKeyword) {
        try {
            tableModel.setColumnCount(0);
            tableModel.setRowCount(0);

            
            StringBuilder selectQuery = new StringBuilder("SELECT ");
            for (int i = 0; i < selectedFields.size(); i++) {
                selectQuery.append(selectedFields.get(i));
                if (i < selectedFields.size() - 1) {
                    selectQuery.append(", ");
                }
            }
            selectQuery.append(" FROM ").append(tableName);
            
            if (!searchKeyword.isEmpty()) {
                selectQuery.append(" WHERE ");
                for (int i = 0; i < selectedFields.size(); i++) {
                    selectQuery.append(selectedFields.get(i)).append(" LIKE '%").append(searchKeyword).append("%'");
                    if (i < selectedFields.size() - 1) {
                        selectQuery.append(" OR ");
                    }
                }
            }

            Statement statement = connection.createStatement();
            ResultSet resultSet = statement.executeQuery(selectQuery.toString());

            
            ResultSetMetaData metaData = resultSet.getMetaData();
            int columnCount = metaData.getColumnCount();
            for (int i = 1; i <= columnCount; i++) {
                tableModel.addColumn(metaData.getColumnName(i));
            }

     
            while (resultSet.next()) {
                Object[] rowData = new Object[columnCount];
                for (int i = 1; i <= columnCount; i++) {
                    rowData[i - 1] = resultSet.getObject(i);
                }
                tableModel.addRow(rowData);
            }

            resultSet.close();
            statement.close();
        } catch (SQLException e) {
            e.printStackTrace();
            JOptionPane.showMessageDialog(null, "Error al realizar la consulta SELECT", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }



 
 private void showUpdateDialog(String tableName) {
     List<String> columnNames = getColumnNames(tableName);

     if (columnNames.isEmpty()) {
         JOptionPane.showMessageDialog(null, "Ez da aldatzeko datuak aurkitu", "Error", JOptionPane.ERROR_MESSAGE);
         return;
     }

     List<JTextField> textFields = new ArrayList<>();

     int selectedRow = table.getSelectedRow();

     if (selectedRow == -1) {
         JOptionPane.showMessageDialog(null, "Sakatu aldatu nahi duzun fila", "Error", JOptionPane.ERROR_MESSAGE);
         return;
     }

     Object[] currentRowData = new Object[columnNames.size()];
     for (int i = 0; i < columnNames.size(); i++) {
         currentRowData[i] = tableModel.getValueAt(selectedRow, i);
     }

     JPanel panel = new JPanel(new GridLayout(0, 2));
     for (int i = 0; i < columnNames.size(); i++) {
         panel.add(new JLabel(columnNames.get(i) + ":"));
         JTextField textField = new JTextField(currentRowData[i].toString(), 20);
         textFields.add(textField);
         panel.add(textField);
     }

     int result = JOptionPane.showConfirmDialog(null, panel, "Aldatu datuak", JOptionPane.OK_CANCEL_OPTION);

     if (result == JOptionPane.OK_OPTION) {
         String[] newValues = textFields.stream().map(JTextField::getText).toArray(String[]::new);

         updateDataInDatabase(tableName, columnNames, currentRowData, newValues);

         for (int i = 0; i < columnNames.size(); i++) {
             tableModel.setValueAt(newValues[i], selectedRow, i);
         }
     }
 }

 private void updateDataInDatabase(String tableName, List<String> columnNames, Object[] currentValues, String[] newValues) {
     try {
         StringBuilder consulta = new StringBuilder("UPDATE " + tableName + " SET ");
         for (int i = 0; i < columnNames.size(); i++) {
             consulta.append(columnNames.get(i)).append(" = ?");
             if (i < columnNames.size() - 1) {
                 consulta.append(", ");
             }
         }
         consulta.append(" WHERE ");

         for (int i = 0; i < columnNames.size(); i++) {
             consulta.append(columnNames.get(i)).append(" = ?");
             if (i < columnNames.size() - 1) {
                 consulta.append(" AND ");
             }
         }

         PreparedStatement preparedStatement = connection.prepareStatement(consulta.toString());

         for (int i = 0; i < newValues.length; i++) {
             preparedStatement.setString(i + 1, newValues[i]);
         }

         for (int i = 0; i < currentValues.length; i++) {
             preparedStatement.setObject(newValues.length + i + 1, currentValues[i]);
         }

         preparedStatement.executeUpdate();
         preparedStatement.close();
     } catch (SQLException e) {
         e.printStackTrace();
         JOptionPane.showMessageDialog(null, "Error aldaketa egiterakoan", "Error", JOptionPane.ERROR_MESSAGE);
     }
 }



    private void selectDataFromDatabase(String tableName) {
        try {
            tableModel.setColumnCount(0);
            tableModel.setRowCount(0);
            String consulta = "SELECT * FROM " + tableName;
            Statement statement = connection.createStatement();
            ResultSet resultSet = statement.executeQuery(consulta);

            tableModel.setRowCount(0);

            ResultSetMetaData metaData = resultSet.getMetaData();
            int columnCount = metaData.getColumnCount();

            for (int i = 1; i <= columnCount; i++) {
                tableModel.addColumn(metaData.getColumnName(i));
            }

            while (resultSet.next()) {
                Object[] rowData = new Object[columnCount];
                for (int i = 1; i <= columnCount; i++) {
                    rowData[i - 1] = resultSet.getObject(i);
                }
                tableModel.addRow(rowData);
            }

            resultSet.close();
            statement.close();
        } catch (SQLException e) {
            e.printStackTrace();
            JOptionPane.showMessageDialog(null, "Error select egiterakoan", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    private void deleteDataFromDatabase(String tableName, String columnName, String idToDelete) {
        try {
            String consulta = "DELETE FROM " + tableName + " WHERE " + columnName + " = ?";
            PreparedStatement preparedStatement = connection.prepareStatement(consulta);
            preparedStatement.setString(1, idToDelete);
            int rowsDeleted = preparedStatement.executeUpdate();
            preparedStatement.close();

            if (rowsDeleted > 0) {
                JOptionPane.showMessageDialog(null, "Erregistroa ezabatu da", "Ezabatu da", JOptionPane.INFORMATION_MESSAGE);
            } else {
                JOptionPane.showMessageDialog(null, "Erregistroa ez da aurkitu", "Aviso", JOptionPane.INFORMATION_MESSAGE);
            }
        } catch (SQLException e) {
            e.printStackTrace();
            JOptionPane.showMessageDialog(null, "Error DELETE operazioan", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }


    private void showInsertDialog(String tableName) {
        List<String> columnNames = getColumnNames(tableName);

        if (columnNames.isEmpty()) {
            JOptionPane.showMessageDialog(null, "Kampoak ez dira aurkitu", "Error", JOptionPane.ERROR_MESSAGE);
            return;
        }

        List<JTextField> textFields = new ArrayList<>();

        JPanel panel = new JPanel(new GridLayout(0, 2));

        for (String columnName : columnNames) {
            panel.add(new JLabel(columnName + ":"));
            JTextField textField = new JTextField(20); 
            textFields.add(textField);
            panel.add(textField);
        }

        int result = JOptionPane.showConfirmDialog(null, panel, "Datuak sartu", JOptionPane.OK_CANCEL_OPTION);

        if (result == JOptionPane.OK_OPTION) {
            String[] values = textFields.stream().map(JTextField::getText).toArray(String[]::new);
            insertDataIntoDatabase(tableName, columnNames, values);
        }
    }

    private List<String> getColumnNames(String tableName) {
        List<String> columnNames = new ArrayList<>();
        try {
            DatabaseMetaData metaData = connection.getMetaData();
            ResultSet resultSet = metaData.getColumns(null, null, tableName, null);

            while (resultSet.next()) {
                String columnName = resultSet.getString("COLUMN_NAME");
                columnNames.add(columnName);
            }

            resultSet.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return columnNames;
    }
    

    private void insertDataIntoDatabase(String tableName, List<String> columnNames, String[] values) {
        try {
            StringBuilder consulta = new StringBuilder("INSERT INTO " + tableName + " (");
            for (int i = 0; i < columnNames.size(); i++) {
                consulta.append(columnNames.get(i));
                if (i < columnNames.size() - 1) {
                    consulta.append(", ");
                }
            }
            consulta.append(") VALUES (");
            for (int i = 0; i < values.length; i++) {
                consulta.append("?");
                if (i < values.length - 1) {
                    consulta.append(", ");
                }
            }
            consulta.append(")");

            PreparedStatement preparedStatement = connection.prepareStatement(consulta.toString());

            for (int i = 0; i < values.length; i++) {
                preparedStatement.setString(i + 1, values[i]);
            }

            preparedStatement.executeUpdate();
            preparedStatement.close();
        } catch (SQLException e) {
            e.printStackTrace();
            JOptionPane.showMessageDialog(null, "Error INSERT egiterakoan", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            Taulak app = new Taulak();
            app.setVisible(true);
        });
    }
}
