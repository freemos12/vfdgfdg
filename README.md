# vfdgfdg

import tkinter as tk
from tkinter import ttk, messagebox
import random
import json
import os

class PasswordGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Password Generator")
        self.root.geometry("500x500")

        # Загрузка истории
        self.history = self.load_history()

        self.setup_ui()

    def setup_ui(self):
        # Ползунок длины пароля
        tk.Label(self.root, text="Длина пароля (8–64):").pack()
        self.length_var = tk.IntVar(value=12)
        length_slider = tk.Scale(self.root, from_=8, to=64, orient=tk.HORIZONTAL,
                               variable=self.length_var)
        length_slider.pack(pady=5)

        # Чекбоксы для выбора символов
        tk.Label(self.root, text="Использовать символы:").pack()

        self.use_digits = tk.BooleanVar(value=True)
        self.use_letters = tk.BooleanVar(value=True)
        self.use_special = tk.BooleanVar(value=False)

        tk.Checkbutton(self.root, text="Цифры (0-9)", variable=self.use_digits).pack()
        tk.Checkbutton(self.root, text="Буквы (a-z, A-Z)", variable=self.use_letters).pack()
        tk.Checkbutton(self.root, text="Спецсимволы (!@#$%)", variable=self.use_special).pack()

        # Кнопка генерации
        tk.Button(self.root, text="Сгенерировать пароль", command=self.generate_password,
                 bg="lightgreen").pack(pady=10)

        # Отображение сгенерированного пароля
        tk.Label(self.root, text="Сгенерированный пароль:").pack()
        self.password_label = tk.Label(self.root, text="", font=("Courier", 12),
                            fg="blue", wraplength=400)
        self.password_label.pack(pady=5)

        # Таблица истории
        tk.Label(self.root, text="История паролей:").pack()
        columns = ("ID", "Пароль", "Длина", "Символы")
        self.history_tree = ttk.Treeview(self.root, columns=columns, show="headings", height=8)

        for col in columns:
            self.history_tree.heading(col, text=col)
            self.history_tree.column(col, width=100)

        self.history_tree.pack(pady=5, fill=tk.BOTH, expand=True)

        # Прокрутка для таблицы
        scrollbar = ttk.Scrollbar(self.root, orient=tk.VERTICAL, command=self.history_tree.yview)
        self.history_tree.configure(yscrollcommand=scrollbar.set)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        self.update_history_display()

    def generate_password(self):
        length = self.length_var.get()

        # Проверка минимальной/максимальной длины
        if length < 8:
            messagebox.showerror("Ошибка", "Минимальная длина пароля — 8 символов!")
            return
        if length > 64:
            messagebox.showerror("Ошибка", "Максимальная длина пароля — 64 символа!")
            return

        # Формирование набора символов
        chars = ""
        if self.use_digits.get():
            chars += "0123456789"
        if self.use_letters.get():
            chars += "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
        if self.use_special.get():
            chars += "!@#$%^&*()_+-=[]{}|;:,.<>?"

        if not chars:
            messagebox.showerror("Ошибка", "Выберите хотя бы один тип символов!")
            return

        # Генерация пароля
        password = ''.join(random.choice(chars) for _ in range(length))
        self.password_label.config(text=password)

        # Добавление в историю
        symbols_used = ""
        if self.use_digits.get(): symbols_used += "Цифры "
        if self.use_letters.get(): symbols_used += "Буквы "
        if self.use_special.get(): symbols_used += "Спецсимволы"

        entry = {
            "password": password,
            "length": length,
            "symbols": symbols_used.strip()
        }
        self.history.append(entry)
        self.save_history()
        self.update_history_display()

    def update_history_display(self):
        for item in self.history_tree.get_children():
            self.history_tree.delete(item)

        for i, entry in enumerate(self.history[-20:], 1):  # Последние 20 записей
            self.history_tree.insert("", tk.END, values=(
                i,
                entry["password"],
                entry["length"],
                entry["symbols"]
            ))

    def save_history(self):
        with open("password_history.json", "w", encoding="utf-8") as f:
            json.dump(self.history, f, ensure_ascii=False, indent=2)

    def load_history(self):
        if os.path.exists("password_history.json"):
            with open("password_history.json", "r", encoding="utf-8") as f:
                return json.load(f)
        return []

if __name__ == "__main__":
    root = tk.Tk()
    app = PasswordGenerator(root)
    root.mainloop()

mkdir random-password-generator
cd random-password-generator
git init

