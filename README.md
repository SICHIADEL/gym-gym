import tkinter as tk
from tkinter import ttk
import sqlite3

class MainWindow(ttk.Frame):
    def __init__(self, parent, *args, **kwargs):
        super().__init__(parent, *args, **kwargs)
        self.parent = parent
        self.setup_main_view()

    def setup_main_view(self):
        # Navigation Buttons
        self.add_member_btn = ttk.Button(self, text="إضافة عضو", command=self.add_member)
        self.add_member_btn.pack(pady=5)

        self.manage_products_btn = ttk.Button(self, text="إدارة المنتجات", command=self.manage_products)
        self.manage_products_btn.pack(pady=5)

        self.manage_subscriptions_btn = ttk.Button(self, text="إدارة الاشتراكات", command=self.manage_subscriptions)
        self.manage_subscriptions_btn.pack(pady=5)

        self.view_reports_btn = ttk.Button(self, text="عرض التقارير", command=self.view_reports)
        self.view_reports_btn.pack(pady=5)

        self.schedule_classes_btn = ttk.Button(self, text="جدولة الفصول", command=self.schedule_classes)
        self.schedule_classes_btn.pack(pady=5)

    def add_member(self):
        AddMemberWindow(self)

    def manage_products(self):
        ManageProductsWindow(self)

    def manage_subscriptions(self):
        ManageSubscriptionsWindow(self)

    def view_reports(self):
        ViewReportsWindow(self)

    def schedule_classes(self):
        ScheduleClassesWindow(self)

class AddMemberWindow(tk.Toplevel):
    def __init__(self, parent):
        super().__init__(parent)
        self.title("إضافة عضو")
        self.geometry("400x300")

        # Name
        ttk.Label(self, text="الاسم:").grid(row=0, column=0, padx=5, pady=5)
        self.name_entry = ttk.Entry(self)
        self.name_entry.grid(row=0, column=1, padx=5, pady=5)

        # Phone
        ttk.Label(self, text="الهاتف:").grid(row=1, column=0, padx=5, pady=5)
        self.phone_entry.grid(row=1, column=1, padx=5, pady=5)

        # Email
        ttk.Label(self, text="البريد الإلكتروني:").grid(row=2, column=0, padx=5, pady=5)
        self.email_entry = ttk.Entry(self)
        self.email_entry.grid(row=2, column=1, padx=5, pady=5)

        # Address
        ttk.Label(self, text="العنوان:").grid(row=3, column=0, padx=5, pady=5)
        self.address_entry = ttk.Entry(self)
        self.address_entry.grid(row=3, column=1, padx=5, pady=5)

        # Membership Start Date
        ttk.Label(self, text="تاريخ بداية الاشتراك:").grid(row=4, column=0, padx=5, pady=5)
        self.start_date_entry = ttk.Entry(self)
        self.start_date_entry.grid(row=4, column=1, padx=5, pady=5)

        # Membership End Date
        ttk.Label(self, text="تاريخ نهاية الاشتراك:").grid(row=5, column=0, padx=5, pady=5)
        self.end_date_entry = ttk.Entry(self)
        self.end_date_entry.grid(row=5, column=1, padx=5, pady=5)

        # Add Button
        self.add_button = ttk.Button(self, text="إضافة", command=self.add_member_to_db)
        self.add_button.grid(row=6, column=0, columnspan=2, padx=5, pady=10)

        # Edit Button
        self.edit_button = ttk.Button(self, text="تعديل", command=self.edit_member)
        self.edit_button.grid(row=7, column=0, columnspan=2, padx=5, pady=10)

        # Listbox to display members
        self.members_listbox = tk.Listbox(self)
        self.members_listbox.grid(row=8, column=0, columnspan=2, padx=5, pady=10)
        self.populate_members_listbox()

    def add_member_to_db(self):
        name = self.name_entry.get()
        phone = self.phone_entry.get()
        email = self.email_entry.get()
        address = self.address_entry.get()
        start_date = self.start_date_entry.get()
        end_date = self.end_date_entry.get()

        # Connect to the database
        conn = sqlite3.connect('gym.db')
        cursor = conn.cursor()

        # Insert the new member
        cursor.execute("""
            INSERT INTO members (name, phone, email, address, membership_start, membership_end)
            VALUES (?, ?, ?, ?, ?, ?)
        """, (name, phone, email, address, start_date, end_date))

        conn.commit()
        conn.close()

        print(f"تم إضافة العضو {name} بنجاح.")
        self.populate_members_listbox()

        # Clear the entry fields
        self.name_entry.delete(0, tk.END)
        self.phone_entry.delete(0, tk.END)
        self.email_entry.delete(0, tk.END)
        self.address_entry.delete(0, tk.END)
        self.start_date_entry.delete(0, tk.END)
        self.end_date_entry.delete(0, tk.END)

    def edit_member(self):
        selected_member_index = self.members_listbox.curselection()
        if selected_member_index:
            selected_member_id = self.members_listbox.get(selected_member_index[0]).split(" - ")[0]
            # Connect to the database
            conn = sqlite3.connect('gym.db')
            cursor = conn.cursor()

            # Get member data
            cursor.execute("""
                SELECT name, phone, email, address, membership_start, membership_end FROM members WHERE member_id = ?
            """, (selected_member_id,))
            member = cursor.fetchone()

            conn.close()

            if member:
                self.name_entry.delete(0, tk.END)
                self.name_entry.insert(0, member[0])
                self.phone_entry.delete(0, tk.END)
                self.phone_entry.insert(0, member[1])
                self.email_entry.delete(0, tk.END)
                self.email_entry.insert(0, member[2])
                self.address_entry.delete(0, tk.END)
                self.address_entry.insert(0, member[3])
                self.start_date_entry.delete(0, tk.END)
                self.start_date_entry.insert(0, member[4])
                self.end_date_entry.delete(0, tk.END)
                self.end_date_entry.insert(0, member[5])
            else:
                print("Member not found.")
        else:
            print("No member selected for editing.")

        # Update Button
        self.update_button = ttk.Button(self, text="تحديث", command=self.update_member_in_db)
        self.update_button.grid(row=9, column=0, columnspan=2, padx=5, pady=10)

    def update_member_in_db(self):
        selected_member_index = self.members_listbox.curselection()
        if selected_member_index:
            selected_member_id = self.members_listbox.get(selected_member_index[0]).split(" - ")[0]
            name = self.name_entry.get()
            phone = self.phone_entry.get()
            email = self.email_entry.get()
            address = self.address_entry.get()
            start_date = self.start_date_entry.get()
            end_date = self.end_date_entry.get()

            # Connect to the database
            conn = sqlite3.connect('gym.db')
            cursor = conn.cursor()

            # Update the member
            cursor.execute("""
                UPDATE members SET name = ?, phone = ?, email = ?, address = ?, membership_start = ?, membership_end = ? WHERE member_id = ?
            """, (name, phone, email, address, start_date, end_date, selected_member_id))

            conn.commit()
            conn.close()

            print(f"تم تحديث العضو {name} بنجاح.")
            self.populate_members_listbox()
        else:
            print("No member selected for updating.")

        # Delete Button
        self.delete_button = ttk.Button(self, text="حذف", command=self.delete_member_from_db)
        self.delete_button.grid(row=10, column=0, columnspan=2, padx=5, pady=10)

    def delete_member_from_db(self):
        selected_member_index = self.members_listbox.curselection()
        if selected_member_index:
            selected_member_id = self.members_listbox.get(selected_member_index[0]).split(" - ")[0]

            # Connect to the database
            conn = sqlite3.connect('gym.db')
            cursor = conn.cursor()

            # Delete the member
            cursor.execute("""
                DELETE FROM members WHERE member_id = ?
            """, (selected_member_id,))

            conn.commit()
            conn.close()

            print(f"تم حذف العضو بنجاح.")
            self.populate_members_listbox()
        else:
            print("No member selected for deleting.")

        # Renew Subscription Button
        self.renew_subscription_button = ttk.Button(self, text="تجديد الاشتراك", command=self.renew_member_subscription)
        self.renew_subscription_button.grid(row=11, column=0, columnspan=2, padx=5, pady=10)

    def renew_member_subscription(self):
        selected_member_index = self.members_listbox.curselection()
        if selected_member_index:
            selected_member_id = self.members_listbox.get(selected_member_index[0]).split(" - ")[0]

            # Connect to the database
            conn = sqlite3.connect('gym.db')
            cursor = conn.cursor()

            # Get the current end date
            cursor.execute("""
                SELECT membership_end FROM members WHERE member_id = ?
            """, (selected_member_id,))
            result = cursor.fetchone()

            if result:
                current_end_date = result[0]
                # Extend the subscription by one month (example)
                new_end_date = self.add_months(current_end_date, 1)

                # Update the membership end date
                cursor.execute("""
                    UPDATE members SET membership_end = ? WHERE member_id = ?
                """, (new_end_date, selected_member_id))

                conn.commit()
                conn.close()

                print(f"تم تجديد اشتراك العضو بنجاح. تاريخ النهاية الجديد: {new_end_date}")
                self.populate_members_listbox()
            else:
                print("Member not found.")
        else:
            print("No member selected for renewing subscription.")

    def add_months(self, start_date, months_to_add):
        from datetime import datetime, timedelta
        from dateutil.relativedelta import relativedelta

        start_date = datetime.strptime(start_date, "%Y-%m-%d")
        new_date = start_date + relativedelta(months=+months_to_add)
        return new_date.strftime("%Y-%m-%d")

    def populate_members_listbox(self):
        # Connect to the database
        conn = sqlite3.connect('gym.db')
        cursor = conn.cursor()

        # Get all members from the database
        cursor.execute("""
            SELECT member_id, name FROM members
        """)
        members = cursor.fetchall()

        conn.close()

        # Clear the listbox
        self.members_listbox.delete(0, tk.END)

        # Add members to the listbox
        for member in members:
            self.members_listbox.insert(tk.END, f"{member[0]} - {member[1]}")

class ManageProductsWindow(tk.Toplevel):
    def __init__(self, parent):
        super().__init__(parent)
        self.title("إدارة المنتجات")
        self.geometry("400x300")

        # Name
        ttk.Label(self, text="الاسم:").grid(row=0, column=0, padx=5, pady=5)
        self.name_entry = ttk.Entry(self)
        self.name_entry.grid(row=0, column=1, padx=5, pady=5)

        # Description
        ttk.Label(self, text="الوصف:").grid(row=1, column=0, padx=5, pady=5)
        self.description_entry = ttk.Entry(self)
        self.description_entry.grid(row=1, column=1, padx=5, pady=5)

        # Price
        ttk.Label(self, text="السعر:").grid(row=2, column=0, padx=5, pady=5)
        self.price_entry = ttk.Entry(self)
        self.price_entry.grid(row=2, column=1, padx=5, pady=5)

        # Stock
        ttk.Label(self, text="المخزون:").grid(row=3, column=0, padx=5, pady=5)
        self.stock_entry = ttk.Entry(self)
        self.stock_entry.grid(row=3, column=1, padx=5, pady=5)

        # Add Button
        self.add_button = ttk.Button(self, text="إضافة", command=self.add_product_to_db)
        self.add_button.grid(row=4, column=0, columnspan=2, padx=5, pady=10)

    def add_product_to_db(self):
        name = self.name_entry.get()
        description = self.description_entry.get()
        price = self.price_entry.get()
        stock = self.stock_entry.get()

        # Connect to the database
        conn = sqlite3.connect('gym.db')
        cursor = conn.cursor()

        # Insert the new product
        cursor.execute("""
            INSERT INTO products (name, description, price, stock)
            VALUES (?, ?, ?, ?)
        """, (name, description, price, stock))

        conn.commit()
        conn.close()

        print(f"تم إضافة المنتج {name} بنجاح.")

        # Clear the entry fields
        self.name_entry.delete(0, tk.END)
        self.description_entry.delete(0, tk.END)
        self.price_entry.delete(0, tk.END)
        self.stock_entry.delete(0, tk.END)

class ManageSubscriptionsWindow(tk.Toplevel):
    def __init__(self, parent):
        super().__init__(parent)
        self.title("إدارة الاشتراكات")
        self.geometry("400x300")

        # Member ID
        ttk.Label(self, text="رقم العضو:").grid(row=0, column=0, padx=5, pady=5)
        self.member_id_entry = ttk.Entry(self)
        self.member_id_entry.grid(row=0, column=1, padx=5, pady=5)

        # Start Date
        ttk.Label(self, text="تاريخ البداية:").grid(row=1, column=0, padx=5, pady=5)
        self.start_date_entry = ttk.Entry(self)
        self.start_date_entry.grid(row=1, column=1, padx=5, pady=5)

        # End Date
        ttk.Label(self, text="تاريخ النهاية:").grid(row=2, column=0, padx=5, pady=5)
        self.end_date_entry = ttk.Entry(self)
        self.end_date_entry.grid(row=2, column=1, padx=5, pady=5)

        # Add Button
        self.add_button = ttk.Button(self, text="إضافة", command=self.add_subscription_to_db)
        self.add_button.grid(row=3, column=0, columnspan=2, padx=5, pady=10)

    def add_subscription_to_db(self):
        member_id = self.member_id_entry.get()
        start_date = self.start_date_entry.get()
        end_date = self.end_date_entry.get()

        # Connect to the database
        conn = sqlite3.connect('gym.db')
        cursor = conn.cursor()

        # Insert the new subscription
        cursor.execute("""
            INSERT INTO subscriptions (member_id, start_date, end_date)
            VALUES (?, ?, ?)
        """, (member_id, start_date, end_date))

        conn.commit()
        conn.close()

        print(f"تم إضافة الاشتراك للعضو {member_id} بنجاح.")

        # Clear the entry fields
        self.member_id_entry.delete(0, tk.END)
        self.start_date_entry.delete(0, tk.END)
        self.end_date_entry.delete(0, tk.END)

class ViewReportsWindow(tk.Toplevel):
    def __init__(self, parent):
        super().__init__(parent)
        self.title("عرض التقارير")
        self.geometry("400x300")

        # Sales Report Button
        self.sales_report_button = ttk.Button(self, text="عرض تقرير المبيعات", command=self.show_sales_report)
        self.sales_report_button.pack(pady=20)

    def show_sales_report(self):
        # Connect to the database
        conn = sqlite3.connect('gym.db')
        cursor = conn.cursor()

        # Get sales data
        cursor.execute("""
            SELECT name, price, stock FROM products
        """)
        products = cursor.fetchall()

        conn.close()

        # Display sales data (for now, just print to console)
        for product in products:
            print(f"المنتج: {product[0]}, السعر: {product[1]}, المخزون: {product[2]}")

class ScheduleClassesWindow(tk.Toplevel):
    def __init__(self, parent):
        super().__init__(parent)
        self.title("جدولة الفصول")
        self.geometry("400x300")

        # Class Name
        ttk.Label(self, text="اسم الفصل:").grid(row=0, column=0, padx=5, pady=5)
        self.name_entry = ttk.Entry(self)
        self.name_entry.grid(row=0, column=1, padx=5, pady=5)

        # Description
        ttk.Label(self, text="الوصف:").grid(row=1, column=0, padx=5, pady=5)
        self.description_entry = ttk.Entry(self)
        self.description_entry.grid(row=1, column=1, padx=5, pady=5)

        # Schedule
        ttk.Label(self, text="الموعد:").grid(row=2, column=0, padx=5, pady=5)
        self.schedule_entry = ttk.Entry(self)
        self.schedule_entry.grid(row=2, column=1, padx=5, pady=5)

        # Instructor
        ttk.Label(self, text="المدرب:").grid(row=3, column=0, padx=5, pady=5)
        self.instructor_entry = ttk.Entry(self)
        self.instructor_entry.grid(row=3, column=1, padx=5, pady=5)

        # Add Button
        self.add_button = ttk.Button(self, text="إضافة", command=self.add_class_to_db)
        self.add_button.grid(row=4, column=0, columnspan=2, padx=5, pady=10)

    def add_class_to_db(self):
        name = self.name_entry.get()
        description = self.description_entry.get()
        schedule = self.schedule_entry.get()
        instructor = self.instructor_entry.get()

        # Connect to the database
        conn = sqlite3.connect('gym.db')
        cursor = conn.cursor()

        # Insert the new class
        cursor.execute("""
            INSERT INTO classes (name, description, schedule, instructor)
            VALUES (?, ?, ?, ?)
        """, (name, description, schedule, instructor))

        conn.commit()
        conn.close()

        print(f"تم إضافة الفصل {name} بنجاح.")

        # Clear the entry fields
        self.name_entry.delete(0, tk.END)
        self.description_entry.delete(0, tk.END)
        self.schedule_entry.delete(0, tk.END)
        self.instructor_entry.delete(0, tk.END)
