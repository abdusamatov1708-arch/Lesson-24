# Lesson-24
class KitobBand(Exception):
    pass


class LimitOshib(Exception):
    pass


class Kitob:
    def __init__(self, sarlavha: str, muallif: str, sahifalar: int):
        self.sarlavha = sarlavha
        self.muallif = muallif
        self.sahifalar = sahifalar  

    @property
    def sahifalar(self):
        return self._sahifalar

    @sahifalar.setter
    def sahifalar(self, value: int):
        if not isinstance(value, int) or value <= 0:
            raise ValueError("Sahifalar soni musbat butun son bo'lishi shart!")
        self._sahifalar = value

    def __eq__(self, other):
        """Kitoblar sarlavha va muallifi bo'yicha tengligini tekshirish"""
        if isinstance(other, Kitob):
            return self.sarlavha == other.sarlavha and self.muallif == other.muallif
        return False

    def __str__(self):
        return f"'{self.sarlavha}' - {self.muallif}"

    def __repr__(self):
        return f"Kitob('{self.sarlavha}', '{self.muallif}', {self.sahifalar})"


# --- Foydalanuvchi Class'i ---
class Foydalanuvchi:
    def __init__(self, ism: str):
        self.ism = ism
        self.olingan_kitoblar = [] 
        self.limit = 3

    def kitob_olish(self, kitob: Kitob):
        if len(self.olingan_kitoblar) >= self.limit:
            raise LimitOshib(f"{self.ism} uchun kitob olish limiti ({self.limit} ta) tugagan!")
        self.olingan_kitoblar.append(kitob)

    def kitob_qaytarish(self, kitob: Kitob):
        if kitob in self.olingan_kitoblar:
            self.olingan_kitoblar.remove(kitob)
        else:
            print(f"{self.ism}da '{kitob.sarlavha}' nomli kitob yo'q.")

    def __str__(self):
        return f"Foydalanuvchi: {self.ism} (Olingan kitoblar: {len(self.olingan_kitoblar)})"

    def __repr__(self):
        return f"Foydalanuvchi('{self.ism}')"


class AVIPUser(Foydalanuvchi):
    def __init__(self, ism: str):
        super().__init__(ism)  
        self.limit = 10  


class Kutubxona:
    def __init__(self, nom: str):
        self.nom = nom
        self.kitoblar = [] 
        self.berilganlar = [] 

    @property
    def mavjud_kitoblar(self):
        """Hozirda javonda turgan kitoblar soni"""
        return len(self.kitoblar)

    @property
    def bandilik_foizi(self):
        """Kitoblarning band bo'lish foizi (computed property)"""
        jami_kitoblar = len(self.kitoblar) + len(self.berilganlar)
        if jami_kitoblar == 0:
            return 0.0
        return (len(self.berilganlar) / jami_kitoblar) * 100

    def kitob_qoshish(self, kitob: Kitob):
        self.kitoblar.append(kitob)

    def kitob_berish(self, kitob: Kitob, foydalanuvchi: Foydalanuvchi):
        if kitob not in self.kitoblar:
            raise KitobBand(f"Kechirasiz, '{kitob.sarlavha}' hozir kutubxonada mavjud emas yoki band!")

        foydalanuvchi.kitob_olish(kitob)  
        self.kitoblar.remove(kitob)  
        self.berilganlar.append(kitob)  

    def kitob_qabul_qilish(self, kitob: Kitob, foydalanuvchi: Foydalanuvchi):
        foydalanuvchi.kitob_qaytarish(kitob)
        if kitob in self.berilganlar:
            self.berilganlar.remove(kitob)
            self.kitoblar.append(kitob)

    def __len__(self):
        """Kutubxonadagi jami kitoblar (mavjud + berilgan)"""
        return len(self.kitoblar) + len(self.berilganlar)

    def __iter__(self):
        """Kutubxonadagi kitoblar bo'ylab iteratsiya qilish"""
        return iter(self.kitoblar)

    def __contains__(self, kitob: Kitob):
        """Kutubxonada (mavjudlar orasida) kitob bor-yo'qligini tekshirish 'in' operatori orqali"""
        return kitob in self.kitoblar

    def __getitem__(self, index: int):
        """Indeks bo'yicha kitobni olish imkoniyati"""
        return self.kitoblar[index]

    def __str__(self):
        return f"Kutubxona: '{self.nom}' (Mavjud: {self.mavjud_kitoblar} ta kitob)"

    def __repr__(self):
        return f"Kutubxona('{self.nom}')"


if __name__ == "__main__":
    k1 = Kitob("Python Dasturlash Asoslari", "T.Aliyev", 350)
    k2 = Kitob("Clean Code", "Robert Martin", 430)
    k3 = Kitob("Design Patterns", "GoF", 395)
    k4 = Kitob("Python Dasturlash Asoslari", "T.Aliyev", 350)  

    print("--- Kitoblar Tengligi ---")
    print(f"k1 == k2: {k1 == k2}")
    print(f"k1 == k4: {k1 == k4}\n")

    lib = Kutubxona("Alisher Navoiy nomidagi kutubxona")
    lib.kitob_qoshish(k1)
    lib.kitob_qoshish(k2)
    lib.kitob_qoshish(k3)

    user1 = Foydalanuvchi("Ali")
    avip_user = AVIPUser("Vali")

    print("--- Kitob Berish ---")
    try:
        lib.kitob_berish(k1, user1)
        print(f"{user1.ism} '{k1.sarlavha}' kitobini oldi.")
    except Exception as e:
        print(f"Xato: {e}")

    print(f"\nMavjud kitoblar: {lib.mavjud_kitoblar} ta")
    print(f"Bandlik foizi: {lib.bandilik_foizi:.1f}%")

    print("\n--- Limitni Tekshirish ---")
    try:
        lib.kitob_berish(k2, user1)
        lib.kitob_berish(k3, user1)
    except LimitOshib as e:
        print(f"Cheklov xatosi: {e}")

    print("\n--- Dunder Metodlar Testi ---")
    print(f"Kutubxonadagi jami kitoblar (len): {len(lib)} ta")
    print(f"Kutubxonada '{k1.sarlavha}' mavjudmi (in): {k1 in lib}")
    print(f"Kutubxona bo'yicha iteratsiya:")
    for kitob in lib:
        print(f" - {kitob}")

    print(f"2-indeksdagi kitob (__getitem__): {lib[1]}")

    print("\n--- __repr__ va __str__ formatlar ---")
    print(str(lib))
    print(repr(k1))
