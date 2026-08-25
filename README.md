import json
from pathlib import Path
from collections import Counter
import re


class SmartNotes:

    def __init__(self):
        self.db = Path("notes.json")

        if not self.db.exists():
            self.db.write_text("[]", encoding="utf-8")

    def load(self):
        with open(self.db, "r", encoding="utf-8") as f:
            return json.load(f)

    def save(self, notes):
        with open(self.db, "w", encoding="utf-8") as f:
            json.dump(notes, f, indent=4, ensure_ascii=False)

    def tokenize(self, text):
        return re.findall(r"\w+", text.lower())

    def add_note(self):
        title = input("Title: ").strip()
        content = input("Content: ").strip()

        notes = self.load()

        notes.append({
            "title": title,
            "content": content
        })

        self.save(notes)

        print("✅ Note saved.")

    def search(self):
        query = input("Search: ").strip()

        words = Counter(self.tokenize(query))

        results = []

        for note in self.load():

            text = note["title"] + " " + note["content"]

            score = 0

            note_words = Counter(self.tokenize(text))

            for word in words:
                score += min(
                    words[word],
                    note_words[word]
                )

            if score:
                results.append((score, note))

        results.sort(reverse=True, key=lambda x: x[0])

        if not results:
            print("No results.")
            return

        print("\nResults\n")

        for score, note in results:

            print("=" * 40)
            print("Score :", score)
            print("Title :", note["title"])
            print("Text  :", note["content"][:120])

    def list_notes(self):

        notes = self.load()

        print()

        for i, note in enumerate(notes, 1):

            print(f"{i}. {note['title']}")


def main():

    app = SmartNotes()

    while True:

        print("\n==== Smart Notes ====")
        print("1. Add Note")
        print("2. Search")
        print("3. List Notes")
        print("4. Exit")

        choice = input("Select: ")

        if choice == "1":
            app.add_note()

        elif choice == "2":
            app.search()

        elif choice == "3":
            app.list_notes()

        elif choice == "4":
            break

        else:
            print("Invalid option")


if __name__ == "__main__":
    main()
