import javafx.application.Application;
import javafx.geometry.Insets;
import javafx.geometry.Pos;
import javafx.scene.Scene;
import javafx.scene.control.*;
import javafx.scene.layout.*;
import javafx.stage.Stage;
import javafx.scene.text.Font;
import javafx.scene.text.FontWeight;
import javafx.scene.paint.Color;
import javafx.scene.chart.*;
import javafx.scene.Node;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

public class ExpenseSplitter extends Application {
	private ListView<String> categoryListView;
	private Map<String, Double> categoryTotals;
	private Button btnAddCategory, btnEditCategory, btnDeleteCategory;
	private VBox categoryManagementPanel;
	private TextField txtAmount, txtTotal, txtCategoryTotal;
	private ArrayList<RadioButton> userButtons;
	private ArrayList<TextField> userAmountFields;
	private VBox panelUsers, panelAmounts;
	private ComboBox<String> categoryBox, splitMethodBox;
	private Map<String, Double> userBalances;
	private TextArea expenseHistoryArea;
	private StringBuilder historyLog;
	private double totalAmount = 0.0;
	//private Map<String, Double> categoryTotals;

	// New chart-related fields
	private BarChart<String, Number> categoryChart;
	private PieChart userBalanceChart;
	private TabPane chartsTabPane;

	public static void main(String[] args) {
		launch(args);
	}
	private VBox createCategoryManagementPanel() {
		categoryManagementPanel = new VBox(15);
		categoryManagementPanel.setPadding(new Insets(20));
		categoryManagementPanel.setStyle("-fx-background-color: white; -fx-effect: dropshadow(three-pass-box, rgba(0,0,0,0.1), 10, 0, 0, 0);");
		categoryManagementPanel.setMinWidth(300);

		Label titleLabel = new Label("Category Management");
		titleLabel.setFont(Font.font("System", FontWeight.BOLD, 20));
		titleLabel.setStyle("-fx-text-fill: #2c3e50;");

		// Create ListView for categories
		categoryListView = new ListView<>();
		categoryListView.setItems(categoryBox.getItems());
		categoryListView.setPrefHeight(200);
		categoryListView.setStyle("-fx-background-color: #f8f9fa; -fx-border-color: #dee2e6; -fx-border-radius: 4;");

		// Create buttons
		HBox buttonBox = new HBox(10);
		buttonBox.setAlignment(Pos.CENTER);

		btnAddCategory = new Button("Add");
		btnEditCategory = new Button("Edit");
		btnDeleteCategory = new Button("Delete");

		styleButton(btnAddCategory, "#3498db");
		styleButton(btnEditCategory, "#f39c12");
		styleButton(btnDeleteCategory, "#e74c3c");

		btnAddCategory.setOnAction(e -> showAddCategoryDialog());
		btnEditCategory.setOnAction(e -> showEditCategoryDialog());
		btnDeleteCategory.setOnAction(e -> deleteSelectedCategory());

		// Disable edit and delete buttons when no category is selected
		btnEditCategory.setDisable(true);
		btnDeleteCategory.setDisable(true);

		// Add selection listener to enable/disable buttons
		categoryListView.getSelectionModel().selectedItemProperty().addListener((obs, oldVal, newVal) -> {
			boolean isSelected = newVal != null;
			btnEditCategory.setDisable(!isSelected);
			btnDeleteCategory.setDisable(!isSelected);
		});

		buttonBox.getChildren().addAll(btnAddCategory, btnEditCategory, btnDeleteCategory);

		// Add all components to the panel
		categoryManagementPanel.getChildren().addAll(titleLabel, categoryListView, buttonBox);

		return categoryManagementPanel;
	}

	// Add category dialog
	private void showAddCategoryDialog() {
		Dialog<String> dialog = new Dialog<>();
		dialog.setTitle("Add Category");
		dialog.setHeaderText("Enter new category name:");

		// Set the button types
		ButtonType addButtonType = new ButtonType("Add", ButtonBar.ButtonData.OK_DONE);
		dialog.getDialogPane().getButtonTypes().addAll(addButtonType, ButtonType.CANCEL);

		// Create the category name field
		TextField categoryNameField = new TextField();
		categoryNameField.setPromptText("Category name");

		// Enable/Disable add button depending on whether text was entered
		Node addButton = dialog.getDialogPane().lookupButton(addButtonType);
		addButton.setDisable(true);

		categoryNameField.textProperty().addListener((observable, oldValue, newValue) -> {
			addButton.setDisable(newValue.trim().isEmpty());
		});

		// Layout
		VBox content = new VBox(10);
		content.getChildren().add(categoryNameField);
		dialog.getDialogPane().setContent(content);

		// Style the dialog
		dialog.getDialogPane().setStyle("-fx-background-color: white;");
		addButton.setStyle("-fx-background-color: #3498db; -fx-text-fill: white;");

		// Convert the result
		dialog.setResultConverter(dialogButton -> {
			if (dialogButton == addButtonType) {
				return categoryNameField.getText().trim();
			}
			return null;
		});

		// Show dialog and process result
		dialog.showAndWait().ifPresent(categoryName -> {
			if (!categoryBox.getItems().contains(categoryName)) {
				categoryBox.getItems().add(categoryName);
				categoryTotals.put(categoryName, 0.0);
				updateCharts();
			} else {
				showAlert("Category already exists!");
			}
		});
	}

	// Edit category dialog
	private void showEditCategoryDialog() {
		String selectedCategory = categoryListView.getSelectionModel().getSelectedItem();
		if (selectedCategory == null) return;

		Dialog<String> dialog = new Dialog<>();
		dialog.setTitle("Edit Category");
		dialog.setHeaderText("Edit category name:");

		ButtonType saveButtonType = new ButtonType("Save", ButtonBar.ButtonData.OK_DONE);
		dialog.getDialogPane().getButtonTypes().addAll(saveButtonType, ButtonType.CANCEL);

		TextField categoryNameField = new TextField(selectedCategory);

		Node saveButton = dialog.getDialogPane().lookupButton(saveButtonType);
		saveButton.setDisable(true);

		categoryNameField.textProperty().addListener((observable, oldValue, newValue) -> {
			saveButton.setDisable(newValue.trim().isEmpty() ||
					newValue.trim().equals(selectedCategory));
		});

		VBox content = new VBox(10);
		content.getChildren().add(categoryNameField);
		dialog.getDialogPane().setContent(content);

		dialog.getDialogPane().setStyle("-fx-background-color: white;");
		saveButton.setStyle("-fx-background-color: #f39c12; -fx-text-fill: white;");

		dialog.setResultConverter(dialogButton -> {
			if (dialogButton == saveButtonType) {
				return categoryNameField.getText().trim();
			}
			return null;
		});

		dialog.showAndWait().ifPresent(newName -> {
			if (!categoryBox.getItems().contains(newName)) {
				int index = categoryBox.getItems().indexOf(selectedCategory);
				categoryBox.getItems().set(index, newName);

				// Update category totals
				Double total = categoryTotals.remove(selectedCategory);
				if (total != null) {
					categoryTotals.put(newName, total);
				}

				// Update current selection if needed
				if (categoryBox.getValue().equals(selectedCategory)) {
					categoryBox.setValue(newName);
				}

				updateCharts();
			} else {
				showAlert("Category name already exists!");
			}
		});
	}

	// Delete category
	private void deleteSelectedCategory() {
		String selectedCategory = categoryListView.getSelectionModel().getSelectedItem();
		if (selectedCategory == null) return;

		Alert confirmDialog = new Alert(Alert.AlertType.CONFIRMATION);
		confirmDialog.setTitle("Delete Category");
		confirmDialog.setHeaderText("Delete category: " + selectedCategory);
		confirmDialog.setContentText("Are you sure you want to delete this category? This action cannot be undone.");

		confirmDialog.getDialogPane().setStyle("-fx-background-color: white;");

		confirmDialog.showAndWait().ifPresent(result -> {
			if (result == ButtonType.OK) {
				categoryBox.getItems().remove(selectedCategory);
				categoryTotals.remove(selectedCategory);

				// If deleted category was selected, select first available category
				if (categoryBox.getValue().equals(selectedCategory)) {
					categoryBox.getSelectionModel().selectFirst();
				}

				updateCharts();
			}
		});
	}

	@Override
	public void start(Stage primaryStage) {
		userButtons = new ArrayList<>();
		userAmountFields = new ArrayList<>();
		userBalances = new HashMap<>();
		historyLog = new StringBuilder();
		categoryTotals = new HashMap<>();

		primaryStage.setTitle("Expense Splitter");

		// Main layout with modern styling
		BorderPane borderPane = new BorderPane();
		borderPane.setStyle("-fx-background-color: #f5f5f5;");

		// Left panel for inputs
		VBox inputPanel = createInputPanel();
		borderPane.setLeft(inputPanel);

		// Center panel for user list and amounts
		VBox centerPanel = createCenterPanel();
		borderPane.setCenter(centerPanel);

		// Right panel for charts
		VBox rightPanel = new VBox(15);
		rightPanel.getChildren().addAll(
				createCategoryManagementPanel(),
				createChartsPanel()
		);
		borderPane.setRight(rightPanel);


		// Bottom panel for expense history
		VBox bottomPanel = createBottomPanel();
		borderPane.setBottom(bottomPanel);

		Scene scene = new Scene(borderPane, 1400, 800);
		primaryStage.setScene(scene);
		primaryStage.show();
	}

	private VBox createChartsPanel() {
		VBox chartsPanel = new VBox(15);
		chartsPanel.setPadding(new Insets(20));
		chartsPanel.setStyle("-fx-background-color: white; -fx-effect: dropshadow(three-pass-box, rgba(0,0,0,0.1), 10, 0, 0, 0);");
		chartsPanel.setMinWidth(400);

		Label chartsTitle = new Label("Expense Analytics");
		chartsTitle.setFont(Font.font("System", FontWeight.BOLD, 20));
		chartsTitle.setStyle("-fx-text-fill: #2c3e50;");

		// Create TabPane for charts
		chartsTabPane = new TabPane();
		chartsTabPane.setTabClosingPolicy(TabPane.TabClosingPolicy.UNAVAILABLE);

		// Create Category Chart
		CategoryAxis xAxis = new CategoryAxis();
		NumberAxis yAxis = new NumberAxis();
		xAxis.setLabel("Categories");
		yAxis.setLabel("Amount ($)");
		categoryChart = new BarChart<>(xAxis, yAxis);
		categoryChart.setTitle("Expenses by Category");
		categoryChart.setAnimated(false);

		// Create User Balance Chart
		userBalanceChart = new PieChart();
		userBalanceChart.setTitle("User Balances");
		userBalanceChart.setLabelsVisible(true);
		userBalanceChart.setAnimated(false);

		// Create tabs and add charts
		Tab categoryTab = new Tab("Categories", categoryChart);
		Tab balanceTab = new Tab("Balances", userBalanceChart);
		chartsTabPane.getTabs().addAll(categoryTab, balanceTab);

		chartsPanel.getChildren().addAll(chartsTitle, chartsTabPane);
		return chartsPanel;
	}

	private VBox createInputPanel() {
		VBox inputPanel = new VBox(15);
		inputPanel.setPadding(new Insets(20));
		inputPanel.setStyle("-fx-background-color: white; -fx-effect: dropshadow(three-pass-box, rgba(0,0,0,0.1), 10, 0, 0, 0);");
		inputPanel.setMinWidth(300);

		// Title
		Label titleLabel = new Label("Expense Details");
		titleLabel.setFont(Font.font("System", FontWeight.BOLD, 20));
		titleLabel.setStyle("-fx-text-fill: #2c3e50;");
		inputPanel.getChildren().add(titleLabel);

		// Amount field
		VBox amountBox = new VBox(5);
		Label amountLabel = new Label("Amount");
		amountLabel.setStyle("-fx-text-fill: #7f8c8d;");
		txtAmount = new TextField("0.00");
		styleTextField(txtAmount);
		amountBox.getChildren().addAll(amountLabel, txtAmount);
		inputPanel.getChildren().add(amountBox);

		// Category selection
		VBox categoryBox = new VBox(5);
		Label categoryLabel = new Label("Category");
		categoryLabel.setStyle("-fx-text-fill: #7f8c8d;");
		this.categoryBox = new ComboBox<>();
		this.categoryBox.getItems().addAll("Food", "Trip", "Utilities", "Other");
		this.categoryBox.getSelectionModel().selectFirst();
		styleComboBox(this.categoryBox);
		categoryBox.getChildren().addAll(categoryLabel, this.categoryBox);
		inputPanel.getChildren().add(categoryBox);

		// Split method selection
		VBox splitMethodBox = new VBox(5);
		Label splitMethodLabel = new Label("Split Method");
		splitMethodLabel.setStyle("-fx-text-fill: #7f8c8d;");
		this.splitMethodBox = new ComboBox<>();
		this.splitMethodBox.getItems().addAll("Equally", "Exact Amount");
		this.splitMethodBox.setOnAction(e -> updateFieldsBasedOnSplitMethod());
		styleComboBox(this.splitMethodBox);
		splitMethodBox.getChildren().addAll(splitMethodLabel, this.splitMethodBox);
		inputPanel.getChildren().add(splitMethodBox);

		// Add User button
		Button btnAddUser = new Button("Add User");
		styleButton(btnAddUser, "#3498db");
		btnAddUser.setOnAction(e -> addUser());
		inputPanel.getChildren().add(btnAddUser);

		// Totals section
		VBox totalsSection = new VBox(10);
		totalsSection.setStyle("-fx-padding: 15 0 0 0; -fx-border-color: #ecf0f1; -fx-border-width: 1 0 0 0;");

		// Total display
		VBox totalBox = new VBox(5);
		Label totalLabel = new Label("Total Expenses");
		totalLabel.setStyle("-fx-text-fill: #7f8c8d;");
		txtTotal = new TextField("0.00");
		txtTotal.setEditable(false);
		styleTextField(txtTotal);
		totalBox.getChildren().addAll(totalLabel, txtTotal);

		// Category total
		VBox categoryTotalBox = new VBox(5);
		Label categoryTotalLabel = new Label("Category Total");
		categoryTotalLabel.setStyle("-fx-text-fill: #7f8c8d;");
		txtCategoryTotal = new TextField("0.00");
		txtCategoryTotal.setEditable(false);
		styleTextField(txtCategoryTotal);
		categoryTotalBox.getChildren().addAll(categoryTotalLabel, txtCategoryTotal);

		totalsSection.getChildren().addAll(totalBox, categoryTotalBox);
		inputPanel.getChildren().add(totalsSection);

		// Action buttons
		HBox buttonBox = new HBox(10);
		buttonBox.setAlignment(Pos.CENTER);
		buttonBox.setPadding(new Insets(20, 0, 0, 0));

		Button btnCalculate = new Button("Calculate");
		styleButton(btnCalculate, "#2ecc71");
		btnCalculate.setOnAction(e -> calculate());

		Button btnReset = new Button("Reset");
		styleButton(btnReset, "#e74c3c");
		btnReset.setOnAction(e -> reset());

		buttonBox.getChildren().addAll(btnCalculate, btnReset);
		inputPanel.getChildren().add(buttonBox);

		return inputPanel;
	}

	private VBox createCenterPanel() {
		VBox centerPanel = new VBox(20);
		centerPanel.setPadding(new Insets(20));
		centerPanel.setStyle("-fx-background-color: white; -fx-effect: dropshadow(three-pass-box, rgba(0,0,0,0.1), 10, 0, 0, 0);");

		Label usersTitle = new Label("Users & Amounts");
		usersTitle.setFont(Font.font("System", FontWeight.BOLD, 20));
		usersTitle.setStyle("-fx-text-fill: #2c3e50;");

		HBox contentBox = new HBox(20);
		panelUsers = new VBox(10);
		panelUsers.setMinWidth(150);
		panelAmounts = new VBox(10);
		panelAmounts.setMinWidth(150);

		Label usersLabel = new Label("Users");
		usersLabel.setStyle("-fx-text-fill: #7f8c8d;");
		Label amountsLabel = new Label("Amounts");
		amountsLabel.setStyle("-fx-text-fill: #7f8c8d;");

		panelUsers.getChildren().add(usersLabel);
		panelAmounts.getChildren().add(amountsLabel);

		contentBox.getChildren().addAll(panelUsers, panelAmounts);
		centerPanel.getChildren().addAll(usersTitle, contentBox);

		return centerPanel;
	}

	private VBox createBottomPanel() {
		VBox bottomPanel = new VBox(10);
		bottomPanel.setPadding(new Insets(20));
		bottomPanel.setStyle("-fx-background-color: white; -fx-effect: dropshadow(three-pass-box, rgba(0,0,0,0.1), 10, 0, 0, 0);");

		Label historyTitle = new Label("Expense History");
		historyTitle.setFont(Font.font("System", FontWeight.BOLD, 20));
		historyTitle.setStyle("-fx-text-fill: #2c3e50;");

		expenseHistoryArea = new TextArea();
		expenseHistoryArea.setEditable(false);
		expenseHistoryArea.setPrefRowCount(5);
		expenseHistoryArea.setStyle("-fx-control-inner-background: #f8f9fa; -fx-border-color: #dee2e6; -fx-border-radius: 4;");

		bottomPanel.getChildren().addAll(historyTitle, expenseHistoryArea);
		return bottomPanel;
	}

	private void updateCharts() {
		// Update Category Chart
		categoryChart.getData().clear();
		XYChart.Series<String, Number> series = new XYChart.Series<>();
		series.setName("Total Expenses");

		for (Map.Entry<String, Double> entry : categoryTotals.entrySet()) {
			series.getData().add(new XYChart.Data<>(entry.getKey(), entry.getValue()));
		}
		categoryChart.getData().add(series);

		// Add hover effect for bar chart
		series.getData().forEach(data -> {
			Node node = data.getNode();
			Tooltip tooltip = new Tooltip(
					String.format("%s: $%.2f", data.getXValue(), data.getYValue())
			);
			Tooltip.install(node, tooltip);
		});

		// Update User Balance Chart
		userBalanceChart.getData().clear();
		for (Map.Entry<String, Double> entry : userBalances.entrySet()) {
			PieChart.Data slice = new PieChart.Data(
					String.format("%s ($%.2f)", entry.getKey(), Math.abs(entry.getValue())),
					Math.abs(entry.getValue())
			);
			userBalanceChart.getData().add(slice);
		}

		// Add hover effect for pie chart
		userBalanceChart.getData().forEach(data -> {
			data.getNode().setOnMouseEntered(e -> {
				data.getNode().setStyle("-fx-pie-color: derive(" + data.getNode().getStyle() + ", -20%);");
			});
			data.getNode().setOnMouseExited(e -> {
				data.getNode().setStyle("");
			});
		});
	}

	private void styleTextField(TextField textField) {
		textField.setStyle("-fx-background-color: #f8f9fa; -fx-border-color: #dee2e6; " +
				"-fx-border-radius: 4; -fx-background-radius: 4; -fx-padding: 8;");
	}

	private void styleComboBox(ComboBox<?> comboBox) {
		comboBox.setStyle("-fx-background-color: #f8f9fa; -fx-border-color: #dee2e6; " +
				"-fx-border-radius: 4; -fx-background-radius: 4;");
		comboBox.setMaxWidth(Double.MAX_VALUE);
	}

	private void styleButton(Button button, String color) {
		button.setStyle("-fx-background-color: " + color + "; -fx-text-fill: white; " +
				"-fx-border-radius: 4; -fx-background-radius: 4; -fx-padding: 8 16;");
		button.setMaxWidth(Double.MAX_VALUE);

		button.setOnMouseEntered(e ->
				button.setStyle("-fx-background-color: derive(" + color + ", -10%); -fx-text-fill: white; " +
						"-fx-border-radius: 4; -fx-background-radius: 4; -fx-padding: 8 16;")
		);
		button.setOnMouseExited(e ->
				button.setStyle("-fx-background-color: " + color + "; -fx-text-fill: white; " +
						"-fx-border-radius: 4; -fx-background-radius: 4; -fx-padding: 8 16;")
		);
	}

	private void updateFieldsBasedOnSplitMethod() {
		String selectedMethod = splitMethodBox.getValue();
		boolean isEqually = "Equally".equals(selectedMethod);

		for (TextField field : userAmountFields) {
			field.setDisable(isEqually);
		}
	}

	private void addUser() {
		TextInputDialog dialog = new TextInputDialog();
		dialog.setTitle("Add User");
		dialog.setHeaderText("Enter user name:");
		dialog.setContentText("User name:");

		dialog.showAndWait().ifPresent(userName -> {
			if (userName.trim().isEmpty() || userBalances.containsKey(userName)) {
				showAlert("User already exists or invalid name.");
				return;
			}

			userBalances.put(userName, 0.0);

			RadioButton userButton = new RadioButton(userName);
			userButton.setStyle("-fx-padding: 5;");
			userButtons.add(userButton);
			panelUsers.getChildren().add(userButton);

			TextField userAmount = new TextField("0.00");
			styleTextField(userAmount);
			userAmount.setDisable(true);
			userAmountFields.add(userAmount);
			panelAmounts.getChildren().add(userAmount);

			// Update charts to reflect new user
			updateCharts();
		});
	}

	private void calculate() {
		double amount;
		try {
			amount = Double.parseDouble(txtAmount.getText());
			if (amount < 0) {
				showAlert("Amount cannot be negative.");
				return;
			}
		} catch (NumberFormatException e) {
			showAlert("Invalid amount entered.");
			return;
		}

		String selectedMethod = splitMethodBox.getValue();
		String selectedCategory = categoryBox.getValue();
		ArrayList<String> involvedUsers = new ArrayList<>();

		if ("Equally".equals(selectedMethod)) {
			int totalPersons = 0;

			for (RadioButton button : userButtons) {
				if (button.isSelected()) {
					totalPersons++;
					involvedUsers.add(button.getText());
				}
			}

			if (totalPersons == 0) {
				showAlert("Please select at least one person.");
				return;
			}

			double amountPerPerson = amount / totalPersons;

			for (int i = 0; i < userButtons.size(); i++) {
				if (userButtons.get(i).isSelected()) {
					String userName = userButtons.get(i).getText();
					userAmountFields.get(i).setText(String.format("%.2f", amountPerPerson));
					double currentBalance = userBalances.get(userName);
					userBalances.put(userName, currentBalance - amountPerPerson);
				}
			}

		} else if ("Exact Amount".equals(selectedMethod)) {
			double totalInput = 0;

			for (int i = 0; i < userButtons.size(); i++) {
				if (userButtons.get(i).isSelected()) {
					String userName = userButtons.get(i).getText();
					double userAmount;
					try {
						userAmount = Double.parseDouble(userAmountFields.get(i).getText());
					} catch (NumberFormatException e) {
						showAlert("Invalid amount for user " + userName);
						return;
					}

					totalInput += userAmount;
					double currentBalance = userBalances.get(userName);
					userBalances.put(userName, currentBalance - userAmount);
					involvedUsers.add(userName);
				}
			}

			if (Math.abs(totalInput - amount) > 0.01) {
				showAlert("The total must match the expense amount.");
				return;
			}
		}

		logExpenseHistory(amount, involvedUsers, selectedCategory);
		updateTotals(amount, selectedCategory);
		updateCharts(); // Update charts after calculation
	}

	private void logExpenseHistory(double amount, ArrayList<String> involvedUsers, String category) {
		StringBuilder logEntry = new StringBuilder();
		logEntry.append(String.format("Amount: $%.2f | ", amount))
				.append("Category: ").append(category)
				.append(" | Users: ").append(String.join(", ", involvedUsers))
				.append("\n");

		// Add individual balances
		logEntry.append("Current Balances:\n");
		for (String user : userBalances.keySet()) {
			logEntry.append(String.format("  %s: $%.2f\n", user, userBalances.get(user)));
		}
		logEntry.append("-".repeat(50)).append("\n");

		historyLog.insert(0, logEntry);  // Add new entries at the top
		expenseHistoryArea.setText(historyLog.toString());
	}

	private void updateTotals(double amount, String selectedCategory) {
		totalAmount += amount;
		txtTotal.setText(String.format("$%.2f", totalAmount));

		// Update the category total
		categoryTotals.put(selectedCategory, categoryTotals.getOrDefault(selectedCategory, 0.0) + amount);
		txtCategoryTotal.setText(String.format("$%.2f", categoryTotals.get(selectedCategory)));
	}

	private void reset() {
		// Clear all user-related data
		userBalances.clear();
		userButtons.clear();
		userAmountFields.clear();

		// Clear user panels
		panelUsers.getChildren().clear();
		panelAmounts.getChildren().clear();

		// Re-add labels for "Users" and "Amounts" after clearing
		Label usersLabel = new Label("Users");
		usersLabel.setStyle("-fx-text-fill: #7f8c8d;");
		panelUsers.getChildren().add(usersLabel);

		Label amountsLabel = new Label("Amounts");
		amountsLabel.setStyle("-fx-text-fill: #7f8c8d;");
		panelAmounts.getChildren().add(amountsLabel);

		// Reset category totals
		for (String category : categoryTotals.keySet()) {
			categoryTotals.put(category, 0.0);
		}

		// Update charts
		updateCharts();

		// Reset input fields
		txtAmount.setText("0.00");
		splitMethodBox.getSelectionModel().selectFirst();
		categoryBox.getSelectionModel().selectFirst();

		// Reset totals
		totalAmount = 0.0;
		txtTotal.setText("0.00");
		txtCategoryTotal.setText("0.00");
		categoryTotals.clear();

		// Clear history
		historyLog.setLength(0);
		expenseHistoryArea.clear();

		// Clear charts
		categoryChart.getData().clear();
		userBalanceChart.getData().clear();

		// Show confirmation dialog
		showConfirmation("Reset Complete", "All users and expenses have been reset.");
	}

	private void showAlert(String message) {
		Alert alert = new Alert(Alert.AlertType.ERROR);
		alert.setTitle("Error");
		alert.setHeaderText(null);
		alert.setContentText(message);

		// Style the alert dialog
		DialogPane dialogPane = alert.getDialogPane();
		dialogPane.setStyle("-fx-background-color: white;");
		dialogPane.getStyleClass().add("modern-dialog");

		// Add custom styling for the buttons
		dialogPane.lookupButton(ButtonType.OK)
				.setStyle("-fx-background-color: #e74c3c; -fx-text-fill: white; " +
						"-fx-border-radius: 4; -fx-background-radius: 4;");

		alert.showAndWait();
	}

	private void showConfirmation(String title, String message) {
		Alert alert = new Alert(Alert.AlertType.INFORMATION);
		alert.setTitle(title);
		alert.setHeaderText(null);
		alert.setContentText(message);

		// Style the confirmation dialog
		DialogPane dialogPane = alert.getDialogPane();
		dialogPane.setStyle("-fx-background-color: white;");
		dialogPane.getStyleClass().add("modern-dialog");

		// Add custom styling for the buttons
		dialogPane.lookupButton(ButtonType.OK)
				.setStyle("-fx-background-color: #2ecc71; -fx-text-fill: white; " +
						"-fx-border-radius: 4; -fx-background-radius: 4;");

		alert.showAndWait();
	}
}
