**1. Raid массивы, что такое и какие бывают:**

*RAID (англ. Redundant Array of Independent Disks — избыточный массив независимых (самостоятельных) дисков)* — технология виртуализации данных для объединения нескольких физических дисковых устройств в логический модуль для повышения отказоустойчивости и (или) производительности.

Петтерсон с коллегами из Беркли представили спецификации пяти уровней RAID, которые стали стандартом де факто:

- RAID 1 — зеркальный дисковый массив;
- RAID 2 — зарезервирован для массивов, которые применяют код Хемминга;
- RAID 3 — дисковый массив с выделенным диском чётности;
- RAID 4 — дисковый массив с чередованием и выделенным диском чётности;
- RAID 5 — дисковый массив с чередованием, в том числе данных чётности (нет диска, выделенного для хранения чётности — блоки чётности чередуются с блоками данных на каждом диске).

Среди современных реализаций массивов RAID представлены дополнительные уровни спецификации:

- RAID 0 — дисковый массив повышенной производительности с чередованием без отказоустойчивости;
- RAID 6 — дисковый массив с чередованием, использующий две контрольные суммы, вычисляемые двумя независимыми способами;
- RAID 10 — массив RAID 0, построенный из массивов RAID 1;
- RAID 50 — массив RAID 0, построенный из массивов RAID 5;
- RAID 60 — массив RAID 0, построенный из массивов RAID 6;
- RAID 1E — зеркальный массив из трёх других массивов: RAID 50, RAID 05, RAID 60 и другие.

---

**2. Добавьте в виртуальную машину 2 диска отформатируйте их в ext4:**

Создание таблиц разделов:

```bash
sudo fdisk /dev/sdb
sudo fdisk /dev/sdc
```

Форматирование дисков в ext4:

```bash
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 /dev/sdc1
```

Проверка:

```bash
blkid
```

![image](https://github.com/user-attachments/assets/e6bc77ba-8efe-4a68-9b8b-683220a85761)


---

**3. Создайте из них raid 0 массив:**

Инициализация RAID 0:

```bash
sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc
```

Создание файловой системы ext4:

```bash
sudo mkfs.ext4 /dev/md0
```

Монтирование RAID массива:

```bash
sudo mkdir -p /mnt/raid0
sudo mount /dev/md0 /mnt/raid0
```

---

**4. Проверье всё ли работает:**

```bash
df -h | grep raid0
```

![image](https://github.com/user-attachments/assets/adbead27-fedd-4f4a-b728-e6825eaf8c82)


---

**5. Удалите raid0 и создайте raid1:**

Остановить массив:

```bash
sudo umount /mnt/raid0
sudo mdadm --stop /dev/md0
```

Удалить массив:

```bash
sudo mdadm --zero-superblock /dev/sdb /dev/sdc
```

Создать RAID 1:

```bash
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```

---

**6. В чём между ними разница?:**

RAID 0 использует чередование, т. е. данные распределяются по всем дискам с высокой скоростью, но без защиты. Однако RAID 1 обеспечивает избыточность за счет зеркального отображения, что означает, что данные записываются одинаково на два диска, что является более безопасным выбором, но объем памяти сокращается вдвое.


---

**7. Есть ли файловые системы которые поддерживают raid массивы без стороненго ПО?**

Да, Btrfs

---

8. Можно ли создать raid массив во время установки системы?

Да