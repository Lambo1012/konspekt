
```python
import re

class DiskpartSimulator:
    """Упрощённый эмулятор diskpart"""
    
    def __init__(self):
        self.current_disk = None
        self.current_partition = None
        self.current_volume = None
        self.disks = {0: {"size": "250 GB", "free": "250 GB", "status": "Online", "gpt": False, "style": "MBR"},
                      1: {"size": "1000 GB", "free": "1000 GB", "status": "Online", "gpt": True, "style": "GPT"},
                      2: {"size": "32 GB", "free": "32 GB", "status": "Offline", "gpt": False, "style": "MBR"}}
        self.partitions = {0: [{"number": 1, "size": "100 MB", "type": "System", "offset": "1024 KB", "active": True}],
                           1: [{"number": 1, "size": "500 GB", "type": "Primary", "offset": "1024 KB", "active": False}]}
        self.volumes = {"C": {"type": "Simple", "size": "200 GB", "status": "Healthy", "fs": "NTFS", "label": "Windows"},
                        "D": {"type": "Simple", "size": "500 GB", "status": "Healthy", "fs": "NTFS", "label": "Data"}}
        self.history = []
    
    def help(self):
        print("""
╔══════════════════════════════════════════╗
║  list disk/partition/volume              ║
║  select disk|partition|volume N          ║
║  detail disk|partition|volume            ║
║  create partition primary/efi size=N     ║
║  delete partition|volume                 ║
║  clean | convert mbr|gpt                 ║
║  format fs=ntfs quick label=X            ║
║  assign letter=X | remove letter         ║
║  active | inactive                       ║
║  online disk | offline disk              ║
║  exit | history | cls                    ║
╚══════════════════════════════════════════╝""")
    
    def list_disks(self):
        print("\n  Диск ###  Состояние  Размер   GPT")
        print("  ---------  ---------  -------  ---")
        for n, d in self.disks.items():
            print(f"  Диск {n}    {d['status']}   {d['size']:>7}  {'*' if d['gpt'] else ' '}")
    
    def list_partitions(self):
        if not self.current_disk: return print("  Сначала выберите диск.")
        print("\n  Раздел ###  Тип          Размер")
        print("  ----------  ------------  --------")
        for p in self.partitions.get(self.current_disk, []):
            print(f"  Раздел {p['number']}    {p['type']:<12} {p['size']:>8}")
    
    def list_volumes(self):
        print("\n  Том ###  Буква  Метка        ФС     Размер")
        print("  -------  -----  -----------  -----  -------")
        for i, (l, v) in enumerate(self.volumes.items(), 1):
            print(f"  Том {i:<5} {l:<6} {v['label']:<11} {v['fs']:<6} {v['size']}")
    
    def select_disk(self, args):
        try:
            n = int(args.split()[-1])
            if n in self.disks:
                self.current_disk = n
                self.current_partition = None
                print(f"  Диск {n} выбран.")
            else:
                print(f"  Диск {n} не найден.")
        except:
            print("  Ошибка: select disk N")
    
    def select_partition(self, args):
        if not self.current_disk: return print("  Сначала выберите диск.")
        try:
            n = int(args.split()[-1])
            for p in self.partitions.get(self.current_disk, []):
                if p["number"] == n:
                    self.current_partition = n
                    print(f"  Раздел {n} выбран.")
                    return
            print(f"  Раздел {n} не найден.")
        except:
            print("  Ошибка: select partition N")
    
    def select_volume(self, args):
        try:
            s = args.split()[-1]
            if s.isalpha() and s.upper() in self.volumes:
                self.current_volume = s.upper()
                print(f"  Том {self.current_volume} выбран.")
            else:
                vols = list(self.volumes.keys())
                if 1 <= int(s) <= len(vols):
                    self.current_volume = vols[int(s)-1]
                    print(f"  Том {self.current_volume} выбран.")
        except:
            print("  Ошибка: select volume N/X")
    
    def create_partition(self, args):
        if not self.current_disk: return print("  Сначала выберите диск.")
        size = re.search(r'size[=\s](\d+)', args)
        size_str = f"{size.group(1)} MB" if size else "100%"
        
        if "primary" in args.lower(): ptype = "Primary"
        elif "efi" in args.lower(): ptype = "EFI System"
        elif "extended" in args.lower(): ptype = "Extended"
        else: return print("  Укажите тип: primary/efi/extended")
        
        if self.current_disk not in self.partitions: self.partitions[self.current_disk] = []
        new_num = len(self.partitions[self.current_disk]) + 1
        self.partitions[self.current_disk].append({"number": new_num, "size": size_str, "type": ptype, "active": False})
        print(f"  Раздел {new_num} создан: {ptype}, {size_str}")
    
    def delete_partition(self):
        if not self.current_partition: return print("  Раздел не выбран.")
        self.partitions[self.current_disk] = [p for p in self.partitions[self.current_disk] if p["number"] != self.current_partition]
        print(f"  Раздел {self.current_partition} удалён.")
        self.current_partition = None
    
    def clean(self):
        if not self.current_disk: return print("  Диск не выбран.")
        if input("  Удалить все разделы? (yes/no): ").lower() == "yes":
            self.partitions[self.current_disk] = []
            print(f"  Диск {self.current_disk} очищен.")
    
    def format_volume(self, args):
        if not self.current_volume: return print("  Том не выбран.")
        fs = re.search(r'fs[=\s](\w+)', args)
        label = re.search(r'label[=\s](\w+)', args)
        self.volumes[self.current_volume]["fs"] = fs.group(1).upper() if fs else "NTFS"
        if label: self.volumes[self.current_volume]["label"] = label.group(1)
        print(f"  Том {self.current_volume} отформатирован в {self.volumes[self.current_volume]['fs']}")
    
    def assign_letter(self, args):
        if not self.current_volume: return print("  Том не выбран.")
        letter = re.search(r'letter[=\s](\w)', args)
        if letter:
            new = letter.group(1).upper()
            self.volumes[new] = self.volumes.pop(self.current_volume)
            self.current_volume = new
            print(f"  Буква {new} назначена.")
    
    def convert(self, args):
        if not self.current_disk: return print("  Диск не выбран.")
        if "gpt" in args.lower():
            self.disks[self.current_disk]["gpt"] = True
            self.disks[self.current_disk]["style"] = "GPT"
            print("  Конвертировано в GPT.")
        elif "mbr" in args.lower():
            self.disks[self.current_disk]["gpt"] = False
            self.disks[self.current_disk]["style"] = "MBR"
            print("  Конвертировано в MBR.")
    
    def run(self):
        self.help()
        print("  Введите 'help' для справки.\n")
        while True:
            try:
                cmd = input("DISKPART> ").strip()
                if not cmd: continue
                self.history.append(cmd)
                c = cmd.lower()
                
                if c in ["exit", "quit"]: break
                elif c == "help" or c == "?": self.help()
                elif c == "list disk": self.list_disks()
                elif c == "list partition": self.list_partitions()
                elif c == "list volume": self.list_volumes()
                elif c.startswith("select disk"): self.select_disk(cmd)
                elif c.startswith("select partition"): self.select_partition(cmd)
                elif c.startswith("select volume"): self.select_volume(cmd)
                elif c.startswith("create partition"): self.create_partition(cmd)
                elif c == "delete partition": self.delete_partition()
                elif c == "clean": self.clean()
                elif c.startswith("format"): self.format_volume(cmd)
                elif c.startswith("assign"): self.assign_letter(cmd)
                elif c.startswith("convert"): self.convert(cmd)
                elif c == "online disk" and self.current_disk: self.disks[self.current_disk]["status"] = "Online"
                elif c == "offline disk" and self.current_disk: self.disks[self.current_disk]["status"] = "Offline"
                elif c == "active" and self.current_partition:
                    for p in self.partitions[self.current_disk]:
                        if p["number"] == self.current_partition: p["active"] = True
                elif c == "history": print("\n" + "\n".join(f"  {i+1}. {h}" for i, h in enumerate(self.history[-10:])))
                elif c == "cls": print("\n" * 50)
                else: print("  Неизвестная команда.")
            except KeyboardInterrupt: break

if __name__ == "__main__":
    DiskpartSimulator().run()
```



