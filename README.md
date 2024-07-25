#include <iostream>
#include <mysql/mysql.h>
#include <sstream>

using namespace std;

const char* hostname = "127.0.0.1";
const char* user = "root";
const char* pass = "123";
const char* dbname = "tokospatu";
unsigned int port = 31235;
const char* unixsocket = NULL;
unsigned long clientflag = 0;

MYSQL* connect_db() {
    MYSQL* conn = mysql_init(0);
    if (conn) {
        conn = mysql_real_connect(conn, hostname, user, pass, dbname, port, unixsocket, clientflag);
        if (conn) {
            cout << "Terhubung ke basis data dengan sukses." << endl;
        } else {
            cerr << "Koneksi gagal: " << mysql_error(conn) << endl;
        }
    } else {
        cerr << "mysql_init gagal" << endl;
    }
    return conn;
}

void add_shoe(const string& name, const string& brand, const string& size, const string& color, int price) {
    MYSQL* conn = connect_db();
    if (conn) {
        stringstream query;
        query << "INSERT INTO sepatu (nama, merk, ukuran, warna, harga) VALUES ('" << name << "', '" << brand << "', '" << size << "', '" << color << "', " << price << ")";
        if (mysql_query(conn, query.str().c_str())) {
            cerr << "INSERT gagal: " << mysql_error(conn) << endl;
        } else {
            cout << "Sepatu berhasil ditambahkan." << endl;
        }
        mysql_close(conn);
    }
}

void show_shoes() {
    MYSQL* conn = connect_db();
    if (conn) {
        if (mysql_query(conn, "SELECT * FROM sepatu")) {
            cerr << "SELECT gagal: " << mysql_error(conn) << endl;
            mysql_close(conn);
            return;
        }

        MYSQL_RES* res = mysql_store_result(conn);
        if (res == nullptr) {
            cerr << "mysql_store_result gagal: " << mysql_error(conn) << endl;
            mysql_close(conn);
            return;
        }

        MYSQL_ROW row;
        while ((row = mysql_fetch_row(res))) {
            cout << "ID: " << row[0] << ", Nama: " << row[1] << ", Merk: " << row[2] << ", Ukuran: " << row[3] << ", Warna: " << row[4] << ", Harga: Rp." << row[5] << endl;
        }

        mysql_free_result(res);
        mysql_close(conn);
    }
}

void update_shoe(int shoe_id, const string& name, const string& brand, const string& size, const string& color, int price) {
    MYSQL* conn = connect_db();
    if (conn) {
        stringstream query;
        query << "UPDATE sepatu SET nama = '" << name << "', merk = '" << brand << "', ukuran = '" << size << "', warna = '" << color << "', harga = " << price << " WHERE id = " << shoe_id;
        if (mysql_query(conn, query.str().c_str())) {
            cerr << "UPDATE gagal: " << mysql_error(conn) << endl;
        } else {
            cout << "Sepatu berhasil diperbarui." << endl;
        }
        mysql_close(conn);
    }
}

void delete_shoe(int shoe_id) {
    MYSQL* conn = connect_db();
    if (conn) {
        stringstream query;
        query << "DELETE FROM sepatu WHERE id = " << shoe_id;
        if (mysql_query(conn, query.str().c_str())) {
            cerr << "DELETE gagal: " << mysql_error(conn) << endl;
        } else {
            cout << "Sepatu berhasil dihapus." << endl;
        }
        mysql_close(conn);
    }
}

void admin_menu() {
    int choice;
    while (true) {
        cout << "\nMenu Admin:\n";
        cout << "1. Tambah Sepatu\n";
        cout << "2. Tampilkan Semua Sepatu\n";
        cout << "3. Perbarui Sepatu\n";
        cout << "4. Hapus Sepatu\n";
        cout << "5. Keluar\n";
        cout << "Masukkan pilihan: ";
        cin >> choice;

        if (choice == 1) {
            string name, brand, size, color;
            int price;
            cout << "Masukkan nama sepatu: ";
            cin.ignore();
            getline(cin, name);
            cout << "Masukkan merk: ";
            getline(cin, brand);
            cout << "Masukkan ukuran: ";
            getline(cin, size);
            cout << "Masukkan warna: ";
            getline(cin, color);
            cout << "Masukkan harga (tanpa titik atau koma): Rp.";
            cin >> price;
            add_shoe(name, brand, size, color, price);
        } else if (choice == 2) {
            show_shoes();
        } else if (choice == 3) {
            int shoe_id, price;
            string name, brand, size, color;
            cout << "Masukkan ID sepatu yang akan diperbarui: ";
            cin >> shoe_id;
            cin.ignore();
            cout << "Masukkan nama baru: ";
            getline(cin, name);
            cout << "Masukkan merk baru: ";
            getline(cin, brand);
            cout << "Masukkan ukuran baru: ";
            getline(cin, size);
            cout << "Masukkan warna baru: ";
            getline(cin, color);
            cout << "Masukkan harga baru (tanpa titik atau koma): Rp.";
            cin >> price;
            update_shoe(shoe_id, name, brand, size, color, price);
        } else if (choice == 4) {
            int shoe_id;
            cout << "Masukkan ID sepatu yang akan dihapus: ";
            cin >> shoe_id;
            delete_shoe(shoe_id);
        } else if (choice == 5) {
            break;
        } else {
            cout << "Pilihan tidak valid. Silakan coba lagi." << endl;
        }
    }
}

void user_menu() {
    int choice;
    while (true) {
        cout << "\nMenu User:\n";
        cout << "1. Tampilkan Semua Sepatu\n";
        cout << "2. Keluar\n";
        cout << "Masukkan pilihan: ";
        cin >> choice;

        if (choice == 1) {
            show_shoes();
        } else if (choice == 2) {
            break;
        } else {
            cout << "Pilihan tidak valid. Silakan coba lagi." << endl;
        }
    }
}

int main() {
    int role_choice;
    cout << "Pilih peran:\n";
    cout << "1. Admin\n";
    cout << "2. User\n";
    cout << "Masukkan pilihan: ";
    cin >> role_choice;

    if (role_choice == 1) {
        admin_menu();
    } else if (role_choice == 2) {
        user_menu();
    } else {
        cout << "Pilihan tidak valid." << endl;
    }

    return 0;
}
