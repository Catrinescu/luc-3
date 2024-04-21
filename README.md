import os
import time
import datetime


class File:
    def __init__(self, filename, created_time, updated_time):
        self.filename = filename
        self.created_time = created_time
        self.updated_time = updated_time

    def info(self):
        print(f"File: {self.filename}")
        print(f"Created Time: {self.created_time}")
        print(f"Last Updated Time: {self.updated_time}")


class TextFile(File):
    def __init__(self, filename, created_time, updated_time, line_count, word_count, char_count):
        super().__init__(filename, created_time, updated_time)
        self.line_count = line_count
        self.word_count = word_count
        self.char_count = char_count

    def info(self):
        super().info()
        print(f"Line Count: {self.line_count}")
        print(f"Word Count: {self.word_count}")
        print(f"Character Count: {self.char_count}")


class ImageFile(File):
    def __init__(self, filename, created_time, updated_time, size):
        super().__init__(filename, created_time, updated_time)
        self.size = size

    def info(self):
        super().info()
        print(f"Size: {self.size}")


class ProgramFile(File):
    def __init__(self, filename, created_time, updated_time, line_count, class_count, method_count):
        super().__init__(filename, created_time, updated_time)
        self.line_count = line_count
        self.class_count = class_count
        self.method_count = method_count

    def info(self):
        super().info()
        print(f"Line Count: {self.line_count}")
        print(f"Class Count: {self.class_count}")
        print(f"Method Count: {self.method_count}")


class FolderMonitor:
    def __init__(self, folder_path):
        self.folder_path = folder_path
        self.files = {}

    def commit(self):
        self.snapshot_time = datetime.datetime.now()

    def monitor_changes(self):
        while True:
            time.sleep(5)  # Check for changes every 5 seconds
            changes_detected = False
            for filename in os.listdir(self.folder_path):
                filepath = os.path.join(self.folder_path, filename)
                if os.path.isfile(filepath):
                    if filename not in self.files:
                        # New file detected
                        self.files[filename] = self.create_file_object(filepath)
                        print(f"New file added: {filename}")
                        changes_detected = True
                    else:
                        # Check if file has been modified
                        updated_time = datetime.datetime.fromtimestamp(os.path.getmtime(filepath))
                        if updated_time > self.files[filename].updated_time:
                            self.files[filename].updated_time = updated_time
                            print(f"File modified: {filename}")
                            changes_detected = True

            # Check for deleted files
            deleted_files = set(self.files.keys()) - set(os.listdir(self.folder_path))
            for filename in deleted_files:
                del self.files[filename]
                print(f"File deleted: {filename}")
                changes_detected = True

            if not changes_detected:
                print("No changes detected.")

    def create_file_object(self, filepath):
        filename, file_extension = os.path.splitext(filepath)
        created_time = datetime.datetime.fromtimestamp(os.path.getctime(filepath))
        updated_time = datetime.datetime.fromtimestamp(os.path.getmtime(filepath))

        if file_extension == ".txt":
            with open(filepath, "r") as f:
                line_count = sum(1 for line in f)
                f.seek(0)
                word_count = sum(len(line.split()) for line in f)
                f.seek(0)
                char_count = sum(len(line) for line in f)
            return TextFile(filename, created_time, updated_time, line_count, word_count, char_count)
        elif file_extension in (".png", ".jpg"):
            size = os.path.getsize(filepath)
            return ImageFile(filename, created_time, updated_time, size)
        elif file_extension == ".py":
            with open(filepath, "r") as f:
                lines = f.readlines()
                line_count = len(lines)
                class_count = sum(1 for line in lines if line.strip().startswith("class "))
                method_count = sum(1 for line in lines if line.strip().startswith("def "))
            return ProgramFile(filename, created_time, updated_time, line_count, class_count, method_count)
        else:
            return File(filename, created_time, updated_time)


if __name__ == "__main__":
    folder_path = "path/to/your/folder"
    folder_monitor = FolderMonitor(folder_path)
    folder_monitor.commit()  # Initial commit
    print("Monitoring changes...")
    try:
        folder_monitor.monitor_changes()
    except KeyboardInterrupt:
        print("Monitoring stopped.")
