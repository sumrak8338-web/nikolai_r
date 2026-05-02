import json
import tkinter as tk
from tkinter import ttk, messagebox, filedialog
from datetime import datetime
from pathlib import Path

class WeatherDiaryApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Weather Diary")
        self.root.geometry("900x600")
        
        # Данные приложения
        self.entries = []          # Список записей
        self.next_id = 1           # Следующий ID для новой записи
        self.current_file = None   # Текущий открытый файл
        
        # Создание интерфейса
        self.create_input_frame()
        self.create_filter_frame()
        self.create_table()
        self.create_button_frame()
        
        # Загрузка данных по умолчанию (если есть)
        self.load_default_file()
        
    def create_input_frame(self):
        """Панель ввода новой записи"""
        input_frame = ttk.LabelFrame(self.root, text="Добавление записи", padding=10)
        input_frame.pack(fill="x", padx=10, pady=5)
        
        # Дата
        ttk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):", font=("Arial", 10)).grid(row=0, column=0, sticky="e", padx=5, pady=5)
        self.date_var = tk.StringVar()
        self.date_entry = ttk.Entry(input_frame, textvariable=self.date_var, width=15, font=("Arial", 10))
        self.date_entry.grid(row=0, column=1, padx=5, pady=5)
        
        # Температура
        ttk.Label(input_frame, text="Температура (°C):", font=("Arial", 10)).grid(row=0, column=2, sticky="e", padx=5, pady=5)
        self.temp_var = tk.StringVar()
        self.temp_entry = ttk.Entry(input_frame, textvariable=self.temp_var, width=10, font=("Arial", 10))
        self.temp_entry.grid(row=0, column=3, padx=5, pady=5)
        
        # Описание
        ttk.Label(input_frame, text="Описание:", font=("Arial", 10)).grid(row=0, column=4, sticky="e", padx=5, pady=5)
        self.desc_var = tk.StringVar()
        self.desc_entry = ttk.Entry(input_frame, textvariable=self.desc_var, width=25, font=("Arial", 10))
        self.desc_entry.grid(row=0, column=5, padx=5, pady=5)
        
        # Осадки
        self.precip_var = tk.BooleanVar()
        self.precip_check = ttk.Checkbutton(input_frame, text="Осадки", variable=self.precip_var, font=("Arial", 10))
        self.precip_check.grid(row=0, column=6, padx=10, pady=5)
        
        # Кнопка добавления
        self.add_btn = ttk.Button(input_frame, text="Добавить запись", command=self.add_entry, width=15)
        self.add_btn.grid(row=0, column=7, padx=10, pady=5)
        
    def create_filter_frame(self):
        """Панель фильтрации"""
        filter_frame = ttk.LabelFrame(self.root, text="Фильтрация записей", padding=10)
        filter_frame.pack(fill="x", padx=10, pady=5)
        
        # Фильтр по дате
        ttk.Label(filter_frame, text="Фильтр по дате:", font=("Arial", 10)).grid(row=0, column=0, sticky="e", padx=5, pady=5)
        self.filter_date_var = tk.StringVar()
        self.filter_date_entry = ttk.Entry(filter_frame, textvariable=self.filter_date_var, width=15, font=("Arial", 10))
        self.filter_date_entry.grid(row=0, column=1, padx=5, pady=5)
        ttk.Button(filter_frame, text="Применить", command=self.apply_filters, width=10).grid(row=0, column=2, padx=5, pady=5)
        
        # Фильтр по температуре
        ttk.Label(filter_frame, text="Температура выше (°C):", font=("Arial", 10)).grid(row=0, column=3, sticky="e", padx=5, pady=5)
        self.filter_temp_var = tk.StringVar()
        self.filter_temp_entry = ttk.Entry(filter_frame, textvariable=self.filter_temp_var, width=10, font=("Arial", 10))
        self.filter_temp_entry.grid(row=0, column=4, padx=5, pady=5)
        ttk.Button(filter_frame, text="Применить", command=self.apply_filters, width=10).grid(row=0, column=5, padx=5, pady=5)
        
        # Кнопка сброса
        ttk.Button(filter_frame, text="Сбросить фильтры", command=self.reset_filters, width=15).grid(row=0, column=6, padx=10, pady=5)
        
    def create_table(self):
        """Создание таблицы для отображения записей"""
        # Фрейм для таблицы и скроллбара
        table_frame = ttk.Frame(self.root)
        table_frame.pack(fill="both", expand=True, padx=10, pady=5)
        
        # Вертикальный скроллбар
        scrollbar = ttk.Scrollbar(table_frame)
        scrollbar.pack(side="right", fill="y")
        
        # Таблица
        columns = ("id", "date", "temperature", "description", "precipitation")
        self.tree = ttk.Treeview(table_frame, columns=columns, show="headings", 
                                 yscrollcommand=scrollbar.set, height=15)
        scrollbar.config(command=self.tree.yview)
        
        # Настройка заголовков
        self.tree.heading("id", text="ID")
        self.tree.heading("date", text="Дата")
        self.tree.heading("temperature", text="Температура (°C)")
        self.tree.heading("description", text="Описание")
        self.tree.heading("precipitation", text="Осадки")
        
        # Настройка ширины колонок
        self.tree.column("id", width=50, anchor="center")
        self.tree.column("date", width=120, anchor="center")
        self.tree.column("temperature", width=100, anchor="center")
        self.tree.column("description", width=300, anchor="w")
        self.tree.column("precipitation", width=80, anchor="center")
        
        self.tree.pack(fill="both", expand=True)
        
    def create_button_frame(self):
        """Нижняя панель с кнопками"""
        button_frame = ttk.Frame(self.root)
        button_frame.pack(fill="x", padx=10, pady=10)
        
        ttk.Button(button_frame, text="Сохранить в JSON", command=self.save_to_file, width=15).pack(side="left", padx=5)
        ttk.Button(button_frame, text="Загрузить из JSON", command=self.load_from_file, width=15).pack(side="left", padx=5)
        ttk.Button(button_frame, text="Удалить выбранную запись", command=self.delete_entry, width=20).pack(side="right", padx=5)
        
        # Статусная строка
        self.status_var = tk.StringVar()
        self.status_var.set("Готово")
        status_label = ttk.Label(button_frame, textvariable=self.status_var, relief="sunken", anchor="w")
        status_label.pack(side="bottom", fill="x", pady=(10, 0))
        
    def is_valid_date(self, date_str):
        """Проверка корректности даты"""
        try:
            datetime.strptime(date_str, "%Y-%m-%d")
            return True
        except ValueError:
            return False
            
    def add_entry(self):
        """Добавление новой записи с проверкой ввода"""
        # Получение данных
        date = self.date_var.get().strip()
        temp_str = self.temp_var.get().strip()
        description = self.desc_var.get().strip()
        precipitation = self.precip_var.get()
        
        # Проверка даты
        if not date:
            messagebox.showerror("Ошибка", "Дата не может быть пустой!")
            return
        if not self.is_valid_date(date):
            messagebox.showerror("Ошибка", "Неверный формат даты!\nИспользуйте: ГГГГ-ММ-ДД (например, 2024-01-15)")
            return
            
        # Проверка температуры
        if not temp_str:
            messagebox.showerror("Ошибка", "Температура не может быть пустой!")
            return
        try:
            temperature = float(temp_str)
        except ValueError:
            messagebox.showerror("Ошибка", "Температура должна быть числом!\nПримеры: 25, -5, 15.5")
            return
            
        # Проверка описания
        if not description:
            messagebox.showerror("Ошибка", "Описание погоды не может быть пустым!")
            return
        
        # Создание записи
        entry = {
            "id": self.next_id,
            "date": date,
            "temperature": temperature,
            "description": description,
            "precipitation": precipitation
        }
        
        self.entries.append(entry)
        self.next_id += 1
        
        # Очистка полей
        self.date_var.set("")
        self.temp_var.set("")
        self.desc_var.set("")
        self.precip_var.set(False)
        
        # Обновление отображения
        self.refresh_display()
        self.update_status(f"Запись добавлена (ID: {entry['id']})")
        messagebox.showinfo("Успех", "Запись успешно добавлена!")
        
    def delete_entry(self):
        """Удаление выбранной записи"""
        selected = self.tree.selection()
        if not selected:
            messagebox.showwarning("Предупреждение", "Пожалуйста, выберите запись для удаления!")
            return
            
        # Подтверждение удаления
        if not messagebox.askyesno("Подтверждение", "Вы уверены, что хотите удалить эту запись?"):
            return
            
        # Получение ID из выбранной строки
        item = self.tree.item(selected[0])
        record_id = item["values"][0]
        
        # Поиск и удаление записи
        for i, entry in enumerate(self.entries):
            if entry["id"] == record_id:
                del self.entries[i]
                break
                
        # Обновление отображения
        self.refresh_display()
        self.update_status(f"Запись с ID {record_id} удалена")
        messagebox.showinfo("Успех", "Запись успешно удалена!")
        
    def apply_filters(self):
        """Применение фильтров"""
        date_filter = self.filter_date_var.get().strip()
        temp_filter_str = self.filter_temp_var.get().strip()
        temp_threshold = None
        
        # Проверка фильтра по дате
        if date_filter:
            if not self.is_valid_date(date_filter):
                messagebox.showerror("Ошибка", "Неверный формат даты в фильтре!\nИспользуйте: ГГГГ-ММ-ДД")
                return
                
        # Проверка фильтра по температуре
        if temp_filter_str:
            try:
                temp_threshold = float(temp_filter_str)
            except ValueError:
                messagebox.showerror("Ошибка", "Фильтр температуры должен быть числом!")
                return
                
        # Очистка таблицы
        for row in self.tree.get_children():
            self.tree.delete(row)
            
        # Применение фильтров и отображение
        filtered_count = 0
        for entry in self.entries:
            # Фильтр по дате
            if date_filter and entry["date"] != date_filter:
                continue
            # Фильтр по температуре
            if temp_threshold is not None and entry["temperature"] <= temp_threshold:
                continue
                
            # Отображение записи
            precip_text = "Да" if entry["precipitation"] else "Нет"
            self.tree.insert("", "end", values=(
                entry["id"],
                entry["date"],
                f"{entry['temperature']:.1f}",
                entry["description"],
                precip_text
            ))
            filtered_count += 1
            
        self.update_status(f"Показано записей: {filtered_count} из {len(self.entries)}")
        
    def reset_filters(self):
        """Сброс всех фильтров"""
        self.filter_date_var.set("")
        self.filter_temp_var.set("")
        self.apply_filters()
        self.update_status("Фильтры сброшены")
        
    def refresh_display(self):
        """Обновление отображения (применение текущих фильтров)"""
        self.apply_filters()
        
    def save_to_file(self):
        """Сохранение записей в JSON файл"""
        if not self.entries:
            messagebox.showwarning("Предупреждение", "Нет записей для сохранения!")
            return
            
        filename = filedialog.asksaveasfilename(
            title="Сохранить дневник погоды",
            defaultextension=".json",
            filetypes=[("JSON files", "*.json"), ("All files", "*.*")],
            initialfile="weather_diary.json"
        )
        
        if filename:
            try:
                with open(filename, "w", encoding="utf-8") as f:
                    json.dump(self.entries, f, ensure_ascii=False, indent=2)
                self.current_file = filename
                self.update_status(f"Данные сохранены в файл: {Path(filename).name}")
                messagebox.showinfo("Успех", f"Данные успешно сохранены в:\n{filename}")
            except Exception as e:
                messagebox.showerror("Ошибка", f"Не удалось сохранить файл:\n{str(e)}")
                
    def load_from_file(self):
        """Загрузка записей из JSON файла"""
        filename = filedialog.askopenfilename(
            title="Загрузить дневник погоды",
            filetypes=[("JSON files", "*.json"), ("All files", "*.*")],
            initialfile="weather_diary.json"
        )
        
        if filename:
            try:
                with open(filename, "r", encoding="utf-8") as f:
                    data = json.load(f)
                    
                if not isinstance(data, list):
                    messagebox.showerror("Ошибка", "Неверный формат файла!")
                    return
                    
                # Проверка структуры данных
                for entry in data:
                    if not all(key in entry for key in ["id", "date", "temperature", "description", "precipitation"]):
                        messagebox.showerror("Ошибка", "Файл содержит некорректные данные!")
                        return
                        
                self.entries = data
                
                # Обновление next_id
                if self.entries:
                    self.next_id = max(entry["id"] for entry in self.entries) + 1
                else:
                    self.next_id = 1
                    
                self.current_file = filename
                self.refresh_display()
                self.update_status(f"Загружено записей: {len(self.entries)} из файла {Path(filename).name}")
                messagebox.showinfo("Успех", f"Загружено {len(self.entries)} записей из:\n{filename}")
                
            except json.JSONDecodeError:
                messagebox.showerror("Ошибка", "Файл не является корректным JSON!")
            except Exception as e:
                messagebox.showerror("Ошибка", f"Не удалось загрузить файл:\n{str(e)}")
                
    def load_default_file(self):
        """Автоматическая загрузка файла по умолчанию при запуске"""
        default_file = "weather_diary.json"
        if Path(default_file).exists():
            try:
                with open(default_file, "r", encoding="utf-8") as f:
                    data = json.load(f)
                if isinstance(data, list):
                    self.entries = data
                    if self.entries:
                        self.next_id = max(entry["id"] for entry in self.entries) + 1
                    self.refresh_display()
                    self.update_status(f"Автоматически загружено {len(self.entries)} записей")
            except Exception:
                pass  # Игнорируем ошибки при автозагрузке
                
    def update_status(self, message):
        """Обновление статусной строки"""
        self.status_var.set(message)
        self.root.after(3000, lambda: self.status_var.set("Готово"))  # Сброс через 3 секунды
        
if __name__ == "__main__":
    root = tk.Tk()
    app = WeatherDiaryApp(root)
    root.mainloop()
