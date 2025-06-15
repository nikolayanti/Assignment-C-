#include <iostream>
#include <fstream>
#include <iomanip>
#include <string>
using namespace std;

const int MaxProducts = 100;

class InventorySystem {
private:
  string ID[MaxProducts];
  string Product_Name[MaxProducts];
  string Category[MaxProducts];
  double Price[MaxProducts];
  int Quantity[MaxProducts];
  int Reorder_level[MaxProducts];
  int Product_count;
  
public:

  // Main menu
  void display_menu() {
    int choice;
    do {
      cout << "\n===== STOCK INVENTORY MANAGEMENT SYSTEM =====" << endl;
      cout << "1. List all products" << endl;
      cout << "2. View a particular product" << endl;
      cout << "3. Add a product" << endl;    
      cout << "4. Delete product(s)" << endl;
      cout << "5. Search product(s)" << endl;
      cout << "6. Update a product" << endl;
      cout << "7. Save and Exit" << endl;
      cout << "Enter your choice: ";
      cin >> choice;
      // Switch to the corresponding operations
      switch(choice) {
        case 1: 
        case 2: 
        case 3:
        case 4:
        case 5:
        case 6:
        case 7:
        default: cout << "Invalid choice! Try again." << endl; 
      }
    } while(choice != 7);
  }
}；
