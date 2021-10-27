## ДЗ-1 майнор Биоинформатика

# Обязательная часть задания
- Создаем папку для выполнения ДЗ-1
    ```
    mkdir hw1
    cd hw1
    ls /usr/share/data-minor-bioinf/assembly/
    ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
    ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
    ```
- Отбираем случайные сэмплы
    ```
    seqtk sample -s302 oil_R1.fastq 5000000 > paired-end_sub1.fastq
    seqtk sample -s302 oil_R2.fastq 5000000 > paired-end_sub2.fastq
    seqtk sample -s302 oilMP_S4_L001_R1_001.fastq 1500000 > mate-pairs_sub1.fastq
    seqtk sample -s302 oilMP_S4_L001_R2_001.fastq 1500000 > mate_pairs_sub2.fastq
    ```
- Считаем статистику с помощью fastqc
    ```
    mkdir fastqc
    ls *sub?.fastq | xargs -t -I % fastqc -o fastqc %
    ```
- Строим отчет с помощью multiqc
    ```
    multiqc -o multiqc fastqc
    ```
- Подрезаем чтения по качеству и удаляем праймеры
    ```
    platanus_trim paired-end_sub1.fastq paired-end_sub2.fastq
    platanus_internal_trim mate-pairs_sub1.fastq mate_pairs_sub2.fastq
    ```
- Собираем multiqc отчет для подрезанных чтений
    ```
    rm -r
    ls -1 *.fastq | xargs -tI{}  {}
    ls *.trimmed | xargs -t -I % fastqc -o fastqc %
    multiqc -o muliqc fastqc
    ```
- Собираем контиги из подрезанных чтений
    ```
    platanus assemble -o Poil -t 1 -m 10 -f *.trimmed 2> platanus_assemble.log
    ```
- Собираем скаффолды из контигов, а также из подрезанных чтений
    ```
    platanus scaffold -o Poil -t 1 -c Poil_contig.fa -IP1 *.trimmed -OP2 *.int_trimmed 2> platanus_scaffold.log
    ```
- Находим самый длинный скаффолд в jupyter notebook и считаем для него количество гэпов и их общую длину

- Закрываем пропуски
    ```
    platanus gap_close -o Poil -t 1 -c Poil_scaffold.fa -IP1 *.trimmed -OP2 *.int_trimmed 2> platanus_gap_close.log
    ```
- Снова находим самый длинный скаффолд в jupyter notebook и считаем для него количество гэпов и их общую длину

# Картинки из отчетов multiqc

multiqc отчеты лежат в корне репозитория в файлах multiqc_report.html и multiqc_report_trimmed.html.

Для исходных чтений
![](/img/source_readings_counts.png)
![](/img/source_readings_general_statistics.png)
![](/img/source_readings_quality.png)

После подрезания чтений по качеству и удаления праймеров
![](/img/trimmed_readings_counts.png)
![](/img/trimmed_readings_general_statistics.png)
![](/img/trimmed_readings_quality.png)


# Весь код лежит в code_hw1.ipynb

# В папке data
Файл contigs.fasta - все полученные контиги

Файл scaffolds.fasta - все полученные скаффолды (после команды gap_close)

Файл longest.fasta - последовательность самого длинного скаффолда (после уменьшения кол-ва гэпов с помощью подрезанных чтений)
