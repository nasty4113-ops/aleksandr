# aleksandr
RandomQuoteGenerator/
│
├── quote_generator.py      # Основной код приложения
├── quotes.json              # Файл для сохранения истории цитат
├── README.md                # Описание проекта
└── .gitignore               # Исключения для Git

import tkinter as tk
from tkinter import ttk, messagebox
import random
import json
import os
from datetime import datetime

class QuoteGenerator:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Quote Generator")
        self.root.geometry("700x600")
        self.root.resizable(True, True)

        # Предопределённые цитаты
        self.predefined_quotes = [
            {"text": "Будь изменением, которое ты хочешь видеть в мире.", "author": "Махатма Ганди", "theme": "Мотивация"},
            {"text": "Жизнь — это то, что с тобой происходит, пока ты строишь планы.", "author": "Джон Леннон", "theme": "Жизнь"},
            {"text": "Воображение важнее знания.", "author": "Альберт Эйнштейн", "theme": "Наука"},
            {"text": "Тот, кто двигает горы, начинает с малых камней.", "author": "Конфуций", "theme": "Мотивация"},
            {"text": "Не бойся совершенства — тебе его не достичь.", "author": "Сальвадор Дали", "theme": "Искусство"},
            {"text": "Сложнее всего начать действовать, всё остальное зависит от упорства.", "author": "Амелия Эрхарт", "theme": "Мотивация"},
            {"text": "Время — лучший учитель, но, к сожалению, оно убивает своих учеников.", "author": "Гектор Берлиоз", "theme": "Жизнь"},
            {"text": "Код — это поэзия.", "author": "Аноним", "theme": "Программирование"}
        ]

        # История сгенерированных цитат
        self.history = []
        self.history_file = "quotes.json"

        # Загружаем историю из файла
        self.load_history()

        # Создаём GUI
        self.create_widgets()

    def create_widgets(self):
        # Рамка для отображения текущей цитаты
        self.quote_frame = tk.LabelFrame(self.root, text="Случайная цитата", padx=10, pady=10)
        self.quote_frame.pack(fill="both", expand=True, padx=10, pady=5)

        self.quote_label = tk.Label(self.quote_frame, text="Нажмите кнопку, чтобы получить цитату", 
                                    wraplength=650, font=("Arial", 12), justify="center")
        self.quote_label.pack(pady=20)

        self.author_label = tk.Label(self.quote_frame, text="", font=("Arial", 10, "italic"))
        self.author_label.pack()

        # Кнопка генерации
        self.generate_btn = tk.Button(self.root, text="Сгенерировать цитату", 
                                      command=self.generate_quote, bg="#4CAF50", fg="white",
                                      font=("Arial", 12), padx=20, pady=5)
        self.generate_btn.pack(pady=10)

        # Рамка для фильтров
        self.filter_frame = tk.LabelFrame(self.root, text="Фильтрация истории", padx=10, pady=5)
        self.filter_frame.pack(fill="x", padx=10, pady=5)

        # Фильтр по автору
        tk.Label(self.filter_frame, text="Автор:").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.author_filter_var = tk.StringVar()
        self.author_filter_combo = ttk.Combobox(self.filter_frame, textvariable=self.author_filter_var, width=25)
        self.author_filter_combo.grid(row=0, column=1, padx=5, pady=5)
        self.author_filter_combo.bind("<<ComboboxSelected>>", lambda e: self.filter_history())

        # Фильтр по теме
        tk.Label(self.filter_frame, text="Тема:").grid(row=0, column=2, padx=5, pady=5, sticky="w")
        self.theme_filter_var = tk.StringVar()
        self.theme_filter_combo = ttk.Combobox(self.filter_frame, textvariable=self.theme_filter_var, width=20)
        self.theme_filter_combo.grid(row=0, column=3, padx=5, pady=5)
        self.theme_filter_combo.bind("<<ComboboxSelected>>", lambda e: self.filter_history())

        # Кнопка сброса фильтров
        self.reset_filter_btn = tk.Button(self.filter_frame, text="Сбросить фильтры", 
                                          command=self.reset_filters)
        self.reset_filter_btn.grid(row=0, column=4, padx=10, pady=5)

        # Рамка для добавления новой цитаты
        self.add_frame = tk.LabelFrame(self.root, text="Добавить новую цитату", padx=10, pady=5)
        self.add_frame.pack(fill="x", padx=10, pady=5)

        tk.Label(self.add_frame, text="Текст цитаты:").grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.new_quote_entry = tk.Text(self.add_frame, height=3, width=50)
        self.new_quote_entry.grid(row=0, column=1, columnspan=3, padx=5, pady=5)

        tk.Label(self.add_frame, text="Автор:").grid(row=1, column=0, padx=5, pady=5, sticky="w")
        self.new_author_entry = tk.Entry(self.add_frame, width=30)
        self.new_author_entry.grid(row=1, column=1, padx=5, pady=5)

        tk.Label(self.add_frame, text="Тема:").grid(row=1, column=2, padx=5, pady=5, sticky="w")
        self.new_theme_entry = tk.Entry(self.add_frame, width=20)
        self.new_theme_entry.grid(row=1, column=3, padx=5, pady=5)

        self.add_quote_btn = tk.Button(self.add_frame, text="+ Добавить цитату в предустановленные", 
                                       command=self.add_new_quote, bg="#2196F3", fg="white")
        self.add_quote_btn.grid(row=2, column=0, columnspan=4, pady=10)

        # Рамка для истории
        self.history_frame = tk.LabelFrame(self.root, text="История сгенерированных цитат", padx=10, pady=5)
        self.history_frame.pack(fill="both", expand=True, padx=10, pady=5)

        # Список истории с прокруткой
        scrollbar = tk.Scrollbar(self.history_frame)
        scrollbar.pack(side="right", fill="y")

        self.history_listbox = tk.Listbox(self.history_frame, yscrollcommand=scrollbar.set, 
                                          font=("Arial", 9), height=12)
        self.history_listbox.pack(fill="both", expand=True)
        scrollbar.config(command=self.history_listbox.yview)

        # Обновляем выпадающие списки фильтров
        self.update_filter_options()
        self.update_history_display()

    def update_filter_options(self):
        """Обновляет доступных авторов и темы для фильтров"""
        authors = sorted(set(quote["author"] for quote in self.history))
        themes = sorted(set(quote["theme"] for quote in self.history))

        self.author_filter_combo["values"] = [""] + authors
        self.theme_filter_combo["values"] = [""] + themes

    def generate_quote(self):
        """Генерирует случайную цитату и добавляет в историю"""
        if not self.predefined_quotes:
            messagebox.showwarning("Нет цитат", "Нет доступных цитат для генерации!")
            return

        quote = random.choice(self.predefined_quotes)
        quote_with_time = {
            **quote,
            "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        }
        self.history.append(quote_with_time)
        
        # Отображаем цитату
        self.quote_label.config(text=f"\"{quote['text']}\"")
        self.author_label.config(text=f"— {quote['author']} (Тема: {quote['theme']})")
        
        # Сохраняем и обновляем
        self.save_history()
        self.update_filter_options()
        self.filter_history()  # Применяем текущие фильтры

    def filter_history(self):
        """Фильтрует историю по выбранным критериям"""
        author_filter = self.author_filter_var.get().strip()
        theme_filter = self.theme_filter_var.get().strip()

        filtered = self.history
        if author_filter:
            filtered = [q for q in filtered if q["author"].lower() == author_filter.lower()]
        if theme_filter:
            filtered = [q for q in filtered if q["theme"].lower() == theme_filter.lower()]

        self.update_history_display(filtered)

    def reset_filters(self):
        """Сбрасывает фильтры"""
        self.author_filter_var.set("")
        self.theme_filter_var.set("")
        self.filter_history()

    def update_history_display(self, filtered_history=None):
        """Обновляет отображение истории в Listbox"""
        self.history_listbox.delete(0, tk.END)
        
        if filtered_history is None:
            filtered_history = self.history

        if not filtered_history:
            self.history_listbox.insert(tk.END, "Нет цитат в истории")
        else:
            for quote in filtered_history:
                display_text = f"[{quote['timestamp']}] {quote['text'][:80]}... — {quote['author']} ({quote['theme']})"
                self.history_listbox.insert(tk.END, display_text)

    def add_new_quote(self):
        """Добавляет новую цитату в предопределённые"""
        text = self.new_quote_entry.get("1.0", tk.END).strip()
        author = self.new_author_entry.get().strip()
        theme = self.new_theme_entry.get().strip()

        # Проверка на пустые строки
        if not text:
            messagebox.showerror("Ошибка", "Текст цитаты не может быть пустым!")
            return
        if not author:
            messagebox.showerror("Ошибка", "Автор не может быть пустым!")
            return
        if not theme:
            messagebox.showerror("Ошибка", "Тема не может быть пустой!")
            return

        new_quote = {"text": text, "author": author, "theme": theme}
        self.predefined_quotes.append(new_quote)
        
        # Очищаем поля ввода
        self.new_quote_entry.delete("1.0", tk.END)
        self.new_author_entry.delete(0, tk.END)
        self.new_theme_entry.delete(0, tk.END)
        
        messagebox.showinfo("Успех", f"Цитата добавлена!\nАвтор: {author}\nТема: {theme}")

    def save_history(self):
        """Сохраняет историю в JSON файл"""
        try:
            with open(self.history_file, 'w', encoding='utf-8') as f:
                json.dump(self.history, f, ensure_ascii=False, indent=2)
        except Exception as e:
            print(f"Ошибка при сохранении истории: {e}")

    def load_history(self):
        """Загружает историю из JSON файла"""
        if os.path.exists(self.history_file):
            try:
                with open(self.history_file, 'r', encoding='utf-8') as f:
                    self.history = json.load(f)
            except Exception as e:
                print(f"Ошибка при загрузке истории: {e}")
                self.history = []
        else:
            self.history = []

if __name__ == "__main__":
    root = tk.Tk()
    app = QuoteGenerator(root)
    root.mainloop()

    [
  {
    "text": "Будь изменением, которое ты хочешь видеть в мире.",
    "author": "Махатма Ганди",
    "theme": "Мотивация",
    "timestamp": "2026-01-15 14:30:22"
  }
]

