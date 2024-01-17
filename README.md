//Code cpp file is in OnlineFoodManagment folder (1st folder) 
#include<iostream>
#include <string>
#include <fstream>
#include <sstream>
#include <iomanip>
using namespace std;
const int MAX_MENU_ITEMS = 100;
// Class for file management
class User;
class FileManager;

class FileManager {
public:
    string userFileName;
    string orderFileName;

    void saveUserToFile(const User& user, string filename);
    void readFromFile(const string& fileName, string data[], int& count);
};

class User
{
public:
    string userID;
    string userName;
    string password;
    string userType;

    void registerUser(string filename, FileManager filemanager) {
        cout << "User Registration:\n";
        cout << "Enter User-ID: ";
        cin >> userID;
        bool Valid = CheckValidity(filename, userID);

        if (Valid)
        {
            cout << "Enter User-Name: ";
            cin >> userName;

            cout << "Set a Password: ";
            cin >> password;


            filemanager.saveUserToFile(*this, filename);
        }
        else
        {
            cout << "ID already Exist!!!\n";
        }
    }
    bool login(string filename) {
        cout << "User Login:\n";
        string UserID, Password;

        cout << "Enter User ID: ";
        cin >> UserID;

        cout << "Enter Password: ";
        cin >> Password;

        bool namefound = false;
        bool idfound = false;

        ifstream file(filename);
        if (!file.is_open()) {
            cout << "Error opening file." << endl;
            return false; // Exit the function if the file cannot be opened
        }

        string row;
        while (getline(file, row)) {
            istringstream ss(row);
            string col;
            int col_num = 0;
            while (getline(ss, col, ',')) {
                if (col_num == 0 && col == UserID) {
                    namefound = true; // Found the UserID in column 0
                }

                if (col_num == 2 && namefound && col == Password) {
                    idfound = true; // Found the Password in column 2 of the same row where UserID was found
                }

                col_num++;
            }

            if (namefound && idfound) {
                break;
            }

            col_num = 0;
        }

        file.close();

        if (namefound && idfound) {
            cout << "Login Successful!\n";
            return true;
        }
        else {
            cout << "Invalid User ID or Password. Login Failed.\n";
        }
    }
    bool CheckValidity(string filename, string userID) {
        bool Idfound = false;

        ifstream ifile(filename);

        string row;
        while (getline(ifile, row))
        {
            istringstream ss(row);
            string col;
            int col_num = 0;
            while (getline(ss, col, ','))
            {
                if (col_num == 0 && col == userID)
                {
                    Idfound = true;
                    if (Idfound)
                    {
                        return false;
                        break;
                    }
                    else
                    {
                        return true;
                    }
                }
            }
        }
        ifile.close();
    }
};
void FileManager::saveUserToFile(const User& user, string filename) {
    ofstream file(filename, ios::app); // Open file in write mode

    if (file.is_open())
    {
        file << user.userID << "," << user.userName << "," << user.password << endl;
        file.close();
    }

    cout << "Registration Successful!\n";
}


class Customer : public User, public FileManager
{
public:
    void viewMenu(string filename) {
        ifstream file(filename);
        string line;
        string items[50][5]; // Array to store item details
        int itemCount = 0;

        cout << "--------------------------------------------------------------------------------------------\n";
        cout << "|    Item Name          |  Price  |  Item ID  |  Quantity  |          Description           |\n";
        cout << "--------------------------------------------------------------------------------------------\n";

        while (getline(file, line) && itemCount < 50) {
            stringstream ss(line);
            int fieldCount = 0;

            while (getline(ss, items[itemCount][fieldCount], ',') && fieldCount < 5) {
                fieldCount++;
            }

            itemCount++;
        }

        cout << setfill(' ') << left;
        for (int i = 0; i < itemCount; ++i) {
            cout << "| " << setw(21) << items[i][0] << " | " << setw(7) << items[i][1] << " | " << setw(9) << items[i][2]
                << " | " << setw(10) << items[i][3] << " | " << setw(28) << items[i][4] << " |\n";
        }
        cout << "--------------------------------------------------------------------------------------------\n";

    }
    void placeOrder(int items, string filename) {
        int* allItems = new int[items];
        int* quan = new int[items];

        for (int i = 0; i < items; i++) {
            cout << "Enter Item ID: ";
            cin >> allItems[i];
            cout << "Enter Quantity: ";
            cin >> quan[i];
        }

        // Read the original CSV file
        ifstream inputFile(filename);
        if (!inputFile.is_open()) {
            cout << "Error opening file." << endl;
            delete[] allItems;
            return;
        }

        ofstream outputFile("Orderhistory.csv");
        if (!outputFile.is_open()) {
            cout << "Error creating Orderhistory.csv." << endl;
            inputFile.close();
            delete[] allItems;
            return;
        }

        string line;
        while (getline(inputFile, line)) {
            istringstream ss(line);
            string itemName, price, id, quantity, description;

            // Extract columns assuming Item Name, Price, ID are in columns 0, 1, and 2 respectively
            getline(ss, itemName, ',');
            getline(ss, price, ',');
            getline(ss, id, ',');
            getline(ss, quantity, ',');
            getline(ss, description);

            int currentItemId = stoi(id);
            for (int i = 0; i < items; ++i) {
                if (currentItemId == allItems[i]) {
                    outputFile << itemName << "," << price << "," << quan[i] << "\n";
                    break;
                }
            }
        }

        inputFile.close();
        outputFile.close();

        cout << "Order placed Successfully:)\n";
        delete[] allItems;
    }
    void viewOrderHistory(FileManager& fileManager) {
        cout << "Viewing Order History...\n";
        // Read order history from the file
        string orderData[100];  // Assuming Max order is defined
        int orderCount = 0;

        fileManager.readFromFile(fileManager.orderFileName, orderData, orderCount);

        if (orderCount > 0)
        {
            cout << "Order History:\n";

            for (int i = 0; i < orderCount; ++i)
            {
                cout << orderData[i] << '\n';
            }
        }
        else {
            cout << "Order history is empty.\n";
        }
    }
    void calculateBill(string filename) {

        // Implementation to view the menu
        ifstream file(filename);
        string line;
        string items[50][5]; // Array to store item details
        int itemCount = 0;

        cout << "----------------------------------------------\n";
        cout << "|    Item Name          |  Price  |  Quantity |\n";
        cout << "----------------------------------------------\n";

        while (getline(file, line) && itemCount < 50) {
            stringstream ss(line);
            int fieldCount = 0;

            while (getline(ss, items[itemCount][fieldCount], ',') && fieldCount < 5) {
                fieldCount++;
            }

            itemCount++;
        }

        cout << setfill(' ') << left;
        for (int i = 0; i < itemCount; ++i) {
            cout << "| " << setw(21) << items[i][0] << " | " << setw(7) << items[i][1] << " | " << setw(9) << items[i][2] << " |\n";
        }
        cout << "----------------------------------------------\n";

    }
};

class MenuItem {
public:
    string itemID;
    string itemName;
    string itemDescription;
    double price;
    int quantityInStock;
};

class CafeStaff : public Customer, public MenuItem {
public:
    void viewMenu(string filename) {
        // Implementation to view the menu
        ifstream file(filename);
        string line;
        string items[50][5]; // Array to store item details
        int itemCount = 0;

        cout << "--------------------------------------------------------------------------------------------\n";
        cout << "|    Item Name          |  Price  |  Item ID  |  Quantity  |          Description           |\n";
        cout << "--------------------------------------------------------------------------------------------\n";

        while (getline(file, line) && itemCount < 50) {
            stringstream ss(line);
            int fieldCount = 0;

            while (getline(ss, items[itemCount][fieldCount], ',') && fieldCount < 5) {
                fieldCount++;
            }

            itemCount++;
        }

        cout << setfill(' ') << left;
        for (int i = 0; i < itemCount; ++i) {
            cout << "| " << setw(21) << items[i][0] << " | " << setw(7) << items[i][1] << " | " << setw(9) << items[i][2]
                << " | " << setw(10) << items[i][3] << " | " << setw(28) << items[i][4] << " |\n";
        }
        cout << "--------------------------------------------------------------------------------------------\n";

    }
    void addMenuItem(string filename) {
        // Implementation to add a menu item
        cout << "Enter the Name of Menu Item to add: ";
        cin >> itemName;
        cout << "Enter the Item ID: ";
        cin >> itemID;
        cout << "Enter the Item Price: ";
        cin >> price;
        cout << "Enter Quantity: ";
        cin >> quantityInStock;
        cout << "Add Description: ";
        cin.ignore();
        getline(cin, itemDescription);

        ofstream file(filename, ios::app);

        if (file.is_open()) {
            file << itemName << "," << price << "," << itemID << "," << quantityInStock << "," << itemDescription << endl;
            file.close();
            cout << "Menu item added successfully!\n";
        }
        else {
            cout << "Unable to open file for writing!\n";
        }
    }

};


int main() {
    FileManager fileManager;

    int ans1,ans2,ans3,ans4;
    string name;
    Customer customer;

    do
    {
        cout << "Define Yourself:\n";
        cout << "1. User\n";
        cout << "2. Admin\n";
        cout << "3. Quit\n";
        cin >> ans1;

        if (ans1 == 1)
        {
            Customer C1;
            do
            {
                cout << "Login OR Sign-UP\n";
                cout << "1. Sign-UP\n";
                cout << "2. Login\n";
                cout << "0. Go Back\n";
                cin >> ans2;

                if (ans2 == 1)
                {
                    C1.registerUser("Student.csv", fileManager);
                    ans2 = 4;
                }
                else if (ans2 == 2)
                {
                    if (C1.login("Student.csv")) {
                        C1.viewMenu("StudentMenu.csv");
                        cout << "Place Order: ";
                        cout << "How many items do you want to order: ";
                        cin >> ans4;
                        C1.placeOrder(ans4, "StudentMenu.csv");
                        C1.calculateBill("Orderhistory.csv");
                    }

                }
                else if (ans2 == 0)
                {
                    ans1 = 0;
                }
                else
                {
                    cout << "Please Enter a valid Input!!!\n";
                }
            } while (ans2 != 0);
        }
        else if (ans1 == 2)
        {
            string name;
            string pass;
            do
            {
                cout << "Press 1 for Cafe Staff\n";
                cout << "Press 0 to Go back\n";
                cin >> ans2;

                if (ans2 == 1)
                {
                    cout << "Login for Cafe Staff:\n";

                    do
                    {
                        cout << "Enter Username: ";
                        cin >> name;
                        if (name == "CStaff")
                        {
                            do
                            {
                                cout << "Enter Password: ";
                                cin >> pass;

                                if (pass == "12345678")
                                {
                                    cout << "Loged-in Successfully:)\n";
                                    CafeStaff Cafe;

                                    do
                                    {
                                        cout << "Choose an option:\n";
                                        cout << "1. See Menu\n";
                                        cout << "2. Add a Menu Item\n";
                                        cout << "0. Go Back\n";

                                        cin >> ans3;

                                        if (ans3 == 1)
                                        {
                                            Cafe.viewMenu("StudentMenu.csv");
                                            ans2 = 4;
                                        }
                                        else if (ans3 == 2)
                                        {
                                            Cafe.addMenuItem("StudentMenu.csv");
                                            ans2 = 4;
                                        }

                                        else if (ans3 == 0)
                                        {
                                            break;
                                        }
                                        else
                                        {
                                            cout << "You entered a Wrong input!!!\n";
                                        }
                                    } while (ans3 != 0);
                                }
                                else
                                {
                                    cout << "Wrong Password!!!\n";
                                }
                            } while (pass != "12345678");
                        }
                        else
                        {
                            cout << "Invalid Username!!!\n";
                        }
                    } while (name != "CStaff");
                }
                else if (ans2 == 0)
                {
                    break;
                }
                else
                {
                    cout << "You entered a wrong input\n";
                }
            } while (ans2 != 0);
        }

        else if (ans1 == 3)
        {
            cout << "Thanks for Visiting:)\n";
            break;
        }
        else
        {
            cout << "Please Enter a valid option!!!\n";
        }


    } while (ans1 != 3);

    return 0;
}
