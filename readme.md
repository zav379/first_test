# Описание

Набор скриптов hdp.coem/*.py - используется для ежедневного цикла отработки данных Game Analytics COEM

# Окружение

В текущем виде скрипты запускается на OS семейства MS WINDOWS - 
c установленным python версии не ниже 3.6 и модулями paramiko, pypyodbc, smtplib, email.mime.text
c доступом к внутренней сети кампании, к 46.165.222.173, 192.168.60.19, 174.129.19.63
при запуске не сервера 192.168.0.33 не экспортируются данные на MS SQL SERVER

# Настройки и алгоритм работы

Настройки расположены в скриптe `settings.py`

pid_file - умалчиваемое имя файла, блокирующего повторный импорт

host - адрес NN HDP для SSH соединения
host_hs2 - адрес ноды c сервисоm HiveServer2 для соединения ODBC
user - имя пользователя Hive HDP
password - имя пользователя Hive HDP
port - порт NN HDP для SSH соединения

ch_params - словарь с настройками разных соединений ClickHouse
list_export_ch - список соединений ClickHouse для экспорта таблиц из MS SQL SERVER
scripts_copy_csv - список команд выполняемых перед импортом CSV в ClickHouse (для ClickHouse в др сетях)
schema_dest_ch - умалчиваемая схема для таблиц импортируемых из MS SQL SERVER в ClickHouse 

csv_folder - словать для папок с ежедневными дампами GAdfz 
csv_extension - расширение для CSV дампов GA
csv_name_date_format - формат даты в имени CSV дампов GA
csv_start_date - начальная дата для сканирования имен дампов GA
START_DUMP - при запросе списка недостающих дампов в таблице (def get_dumps - utils.py)
    и отсутствии данных о последнем дампе в таблице такая дата из будущего формирует пустого списков дампов 
    Если таблица пустая, то эту переменную можно заменить на начальную дату данных для этой таблицы минус 1 день
    и функция выдаст список дат с начальной даты данных по вчерашнюю

CSV_SQL2_PATH - папка для обмена CSV
PLACE_IN_SQL4 - база и схема по умолчанию для таблиц, импортируемых на MS SQL SERVER
MOUNT_DIRS - словарь соответствий монтируемых сетевых папок в Linux
dir_exp_hdp - внутренняя папка в Linux для экспорта в CSV
ROOT_DIR_HDP - монтированная папка в HDP c дампами GA
ROOT_DIR_WIN - сетевая папка c дампами GA (для Windows)
DIR_CURRENCY_RATES - папка с курсами валют (доступна только на 192.168.0.33)
LTV_DIR - папка для данных LTV 
IDFA_EXP_DIR - папка для данных IDFA на DropBox

db_name_rstat - префикс схем на HDP для GA
db_name_ads - префикс схема на HDP для данных AF, импортируемых  из 192.168.0.33
table_ads - таблица инсталлов AF на HDP 

log_filename - имя основного лога по умолчанию
log_current_filename - имя лога за последнюю дату по умолчанию
log_error - имя лога ошибок

mssql_connection_string - настройки ODBC для MS SQL SERVER
report_affected_versions - словарь версий COEM
report_updated_versions - словарь версий, с каких обновляются репорты из модуля reports.py
two_stage_import - словарь итераций для попыток импорта, начиная с которых данные основной версии 
   импортируются отдельно от остального дампа
ua_platforms - словарь перевода имен платформ на UA-версии
cohort_default - когорты по умолчанию
cohort_begin - универсальная когорта по умолчанию
calc_payer_group - формула определения групп плательщиков
Max* - ограничение максимальных значений для аггрегированных репортов из reports.py
mail_* - параметры почты
slack_to - список рассылки в Slack по умолчанию
slack_to_Hive2 - список рассылки в Slack по сбоям HDP JDBC
slack_to_new_pack - список рассылки в Slack по новым пакам кристаллов

uploading_ftp_list - список файлов для экспорта на FTP
export_ftp_* - параметры FTP для для экспорта CSV
export_users_query - скрипт для экспорта пользователей на FTP
char_* - константы символов для Linux через Python, конфликтующими с регулярнями символами

exp_segm - условия для разбивки больших CSV для экспорта
exp_segm_tables - список разбиваемых таблиц для экспорта CSV 

def platform_list - список актуальных платформ
def csv_path - список путей к дампам GA

monitor_logs_max_delay - интевал отсутствия логирования, после которого идет нотификация в Slack
monitor_logs_interval - интевал проверки связи c HDP по JDBC
def get_arg_p - собирает набор аргументов экспорта MSSQL в Clickhouse из командной строки

ITERATION_INTERVALS - временные паузы при сбое импорта CSV в Clickhouse 
wait_af - время ожидания инсталлов из AF (при необновлении используются вчерашние данные)
check_python - признак тастирования Python -скриптов 
    (если True - скрипты выполняются очень быстро, тк действий с данными не происходит)
EXPORT_EXCLUSION - список таблиц, где не допускается стандартный экспорт MSSQL в Clickhouse 

token_serv_val - токен для сервера валидации платежей
CSV_SERV_VAL - CSV для выгрузки данных валидации платежей
PLATFORMS_SERV_VAL - платформы в данных валидации платежей

SLA_CH - список SLA таблиц UA, экспортируются в Clickhouse 
CURR_TZ - текущая таймзона для сервера HDP

# Логирование и нотификация

Предусмотрено 3 механизма:
- логирование в текстовые файлы текущей папки *.log;
- стандартный консольный вывод, при интерактивном запуске;
- Slack-нотификации

# Запуск
    awem_analitics.py {platform} report или start_platform.cmd {platform} 
	- цикл отработки данных Game Analytics COEM (импорт - репорты)
    export.py Export_SQL_CH {таблица} - экспорт MSSQL в Clickhouse c верификацией
    export.py export_idfa - экспорт IDFA на DropBox 
    utils.py monitor_logs - запуск монитора, отслеживающего зависания awem_analitics.py {platform} report 
       проверяющего JDBC HDP и обновление UA SLA на Clickhouse 	
    utils.py slack_notification {текст} - нотификация в SLACK 
    

