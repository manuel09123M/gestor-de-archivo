import os
import shutil
from datetime import datetime
import tkinter as tk
from tkinter import filedialog, messagebox, ttk

class FileManagerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Gestor de Archivos")
        self.root.geometry("800x600")
        
        # Variables
        self.current_dir = tk.StringVar(value=os.getcwd())
        self.backup_dir = tk.StringVar(value=os.path.join(os.getcwd(), "backups"))
        
        # Crear interfaz
        self.create_widgets()
        
        # Actualizar lista de archivos
        self.update_file_list()
        
    def create_widgets(self):
        # Frame superior (barra de herramientas)
        toolbar_frame = tk.Frame(self.root, bd=1, relief=tk.RAISED)
        toolbar_frame.pack(fill=tk.X, padx=2, pady=2)
        
        # Botones de la barra de herramientas
        tk.Button(toolbar_frame, text="Crear Archivo", command=self.create_file).pack(side=tk.LEFT, padx=2)
        tk.Button(toolbar_frame, text="Leer Archivo", command=self.read_file).pack(side=tk.LEFT, padx=2)
        tk.Button(toolbar_frame, text="Editar Archivo", command=self.edit_file).pack(side=tk.LEFT, padx=2)
        tk.Button(toolbar_frame, text="Eliminar Archivo", command=self.delete_file).pack(side=tk.LEFT, padx=2)
        tk.Button(toolbar_frame, text="Espacio en Disco", command=self.show_disk_space).pack(side=tk.LEFT, padx=2)
        tk.Button(toolbar_frame, text="Respaldar Archivos", command=self.backup_files).pack(side=tk.LEFT, padx=2)
        
        # Frame de navegación
        nav_frame = tk.Frame(self.root)
        nav_frame.pack(fill=tk.X, padx=5, pady=5)
        
        tk.Label(nav_frame, text="Directorio Actual:").pack(side=tk.LEFT)
        tk.Entry(nav_frame, textvariable=self.current_dir, width=50).pack(side=tk.LEFT, padx=5)
        tk.Button(nav_frame, text="Cambiar", command=self.change_directory).pack(side=tk.LEFT, padx=2)
        tk.Button(nav_frame, text="Refrescar", command=self.update_file_list).pack(side=tk.LEFT, padx=2)
        
        # Frame de lista de archivos
        list_frame = tk.Frame(self.root)
        list_frame.pack(fill=tk.BOTH, expand=True, padx=5, pady=5)
        
        # Treeview para mostrar archivos
        self.tree = ttk.Treeview(list_frame, columns=('Name', 'Size', 'Type', 'Modified'), selectmode='browse')
        self.tree.heading('#0', text='')
        self.tree.heading('Name', text='Nombre')
        self.tree.heading('Size', text='Tamaño')
        self.tree.heading('Type', text='Tipo')
        self.tree.heading('Modified', text='Modificado')
        
        self.tree.column('#0', width=0, stretch=tk.NO)
        self.tree.column('Name', width=200, anchor=tk.W)
        self.tree.column('Size', width=100, anchor=tk.E)
        self.tree.column('Type', width=100, anchor=tk.W)
        self.tree.column('Modified', width=150, anchor=tk.W)
        
        scrollbar = ttk.Scrollbar(list_frame, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscrollcommand=scrollbar.set)
        
        self.tree.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        
        # Barra de estado
        self.status_bar = tk.Label(self.root, text="Listo", bd=1, relief=tk.SUNKEN, anchor=tk.W)
        self.status_bar.pack(fill=tk.X)
        
    def update_file_list(self):
        """Actualiza la lista de archivos en el directorio actual"""
        self.tree.delete(*self.tree.get_children())
        current_dir = self.current_dir.get()
        
        try:
            # Agregar directorio padre (..)
            if current_dir != os.path.dirname(current_dir):
                self.tree.insert('', 'end', values=('..', '', 'Directorio', ''))
            
            # Listar archivos y directorios
            for item in os.listdir(current_dir):
                full_path = os.path.join(current_dir, item)
                if os.path.isdir(full_path):
                    item_type = "Directorio"
                    size = ""
                else:
                    item_type = "Archivo"
                    size = self.get_human_readable_size(os.path.getsize(full_path))
                
                modified = datetime.fromtimestamp(os.path.getmtime(full_path)).strftime('%Y-%m-%d %H:%M:%S')
                self.tree.insert('', 'end', values=(item, size, item_type, modified))
                
        except Exception as e:
            messagebox.showerror("Error", f"No se pudo listar el directorio: {e}")
    
    def get_human_readable_size(self, size):
        """Convierte el tamaño de bytes a un formato legible"""
        for unit in ['B', 'KB', 'MB', 'GB']:
            if size < 1024.0:
                return f"{size:.1f} {unit}"
            size /= 1024.0
        return f"{size:.1f} TB"
    
    def change_directory(self):
        """Cambia el directorio actual"""
        new_dir = filedialog.askdirectory(initialdir=self.current_dir.get())
        if new_dir:
            self.current_dir.set(new_dir)
            self.update_file_list()
    
    def create_file(self):
        """Crea un nuevo archivo"""
        file_name = simpledialog.askstring("Crear Archivo", "Nombre del archivo:")
        if file_name:
            try:
                file_path = os.path.join(self.current_dir.get(), file_name)
                with open(file_path, 'w') as f:
                    f.write("")
                self.update_file_list()
                self.status_bar.config(text=f"Archivo creado: {file_name}")
            except Exception as e:
                messagebox.showerror("Error", f"No se pudo crear el archivo: {e}")
    
    def read_file(self):
        """Lee el contenido de un archivo"""
        selected = self.get_selected_item()
        if selected:
            file_path = os.path.join(self.current_dir.get(), selected)
            if os.path.isfile(file_path):
                try:
                    with open(file_path, 'r') as f:
                        content = f.read()
                    
                    # Mostrar contenido en una nueva ventana
                    content_window = tk.Toplevel(self.root)
                    content_window.title(f"Contenido de {selected}")
                    
                    text_widget = tk.Text(content_window, wrap=tk.WORD)
                    scrollbar = tk.Scrollbar(content_window, command=text_widget.yview)
                    text_widget.configure(yscrollcommand=scrollbar.set)
                    
                    text_widget.insert(tk.END, content)
                    text_widget.config(state=tk.DISABLED)
                    
                    text_widget.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
                    scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
                    
                except Exception as e:
                    messagebox.showerror("Error", f"No se pudo leer el archivo: {e}")
            else:
                messagebox.showwarning("Advertencia", "Seleccione un archivo, no un directorio")
    
    def edit_file(self):
        """Edita el contenido de un archivo"""
        selected = self.get_selected_item()
        if selected:
            file_path = os.path.join(self.current_dir.get(), selected)
            if os.path.isfile(file_path):
                try:
                    with open(file_path, 'r') as f:
                        content = f.read()
                    
                    # Ventana de edición
                    edit_window = tk.Toplevel(self.root)
                    edit_window.title(f"Editando {selected}")
                    
                    text_widget = tk.Text(edit_window, wrap=tk.WORD)
                    scrollbar = tk.Scrollbar(edit_window, command=text_widget.yview)
                    text_widget.configure(yscrollcommand=scrollbar.set)
                    
                    text_widget.insert(tk.END, content)
                    
                    text_widget.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
                    scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
                    
                    # Botón para guardar
                    def save_changes():
                        try:
                            with open(file_path, 'w') as f:
                                f.write(text_widget.get("1.0", tk.END))
                            messagebox.showinfo("Éxito", "Archivo guardado correctamente")
                            edit_window.destroy()
                        except Exception as e:
                            messagebox.showerror("Error", f"No se pudo guardar el archivo: {e}")
                    
                    save_button = tk.Button(edit_window, text="Guardar", command=save_changes)
                    save_button.pack(pady=5)
                    
                except Exception as e:
                    messagebox.showerror("Error", f"No se pudo editar el archivo: {e}")
            else:
                messagebox.showwarning("Advertencia", "Seleccione un archivo, no un directorio")
    
    def delete_file(self):
        """Elimina un archivo o directorio"""
        selected = self.get_selected_item()
        if selected:
            file_path = os.path.join(self.current_dir.get(), selected)
            confirm = messagebox.askyesno("Confirmar", f"¿Está seguro que desea eliminar {selected}?")
            if confirm:
                try:
                    if os.path.isfile(file_path):
                        os.remove(file_path)
                    else:
                        shutil.rmtree(file_path)
                    self.update_file_list()
                    self.status_bar.config(text=f"Eliminado: {selected}")
                except Exception as e:
                    messagebox.showerror("Error", f"No se pudo eliminar: {e}")
    
    def show_disk_space(self):
        """Muestra el espacio disponible en disco"""
        total, used, free = shutil.disk_usage(self.current_dir.get())
        
        disk_info = f"""
        Espacio en disco para {self.current_dir.get()}:
        
        Total: {self.get_human_readable_size(total)}
        Usado: {self.get_human_readable_size(used)} ({used/total*100:.1f}%)
        Libre: {self.get_human_readable_size(free)} ({free/total*100:.1f}%)
        """
        
        messagebox.showinfo("Espacio en Disco", disk_info)
    
    def backup_files(self):
        """Crea una copia de seguridad de los archivos seleccionados"""
        selected = self.get_selected_item()
        if not selected:
            messagebox.showwarning("Advertencia", "Seleccione un archivo o directorio para respaldar")
            return
        
        source_path = os.path.join(self.current_dir.get(), selected)
        backup_path = filedialog.askdirectory(
            initialdir=self.backup_dir.get(),
            title="Seleccione directorio de respaldo"
        )
        
        if backup_path:
            self.backup_dir.set(backup_path)
            try:
                timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
                backup_name = f"{selected}_backup_{timestamp}"
                dest_path = os.path.join(backup_path, backup_name)
                
                if os.path.isfile(source_path):
                    shutil.copy2(source_path, dest_path)
                else:
                    shutil.copytree(source_path, dest_path)
                
                messagebox.showinfo("Éxito", f"Respaldo creado en:\n{dest_path}")
                self.status_bar.config(text=f"Respaldo creado: {backup_name}")
            except Exception as e:
                messagebox.showerror("Error", f"No se pudo crear el respaldo: {e}")
    
    def get_selected_item(self):
        """Obtiene el elemento seleccionado en el treeview"""
        selected = self.tree.focus()
        if selected:
            return self.tree.item(selected)['values'][0]
        messagebox.showwarning("Advertencia", "No se ha seleccionado ningún archivo o directorio")
        return None

# Función para importar simpledialog si no está disponible
try:
    from tkinter import simpledialog
except ImportError:
    import tkinter.simpledialog as simpledialog

# Iniciar la aplicación
if __name__ == "__main__":
    root = tk.Tk()
    app = FileManagerApp(root)
    root.mainloop()
