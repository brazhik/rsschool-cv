# **Дмитрий Бражников**

## **Контакты**
- **E-mail:** da.brazhnikov@gmail.com
- **Телефон:** +7-XXX-XXX-XX-XX

## **Краткая информация о себе**
- **Цель и приоритеты:** погрузиться во фронтенд-разработку (React);
- **Текущая специализация:** являюсь разработчиком на языке ABAP для продуктов компании SAP;
- **Сильные стороны:** внимательность, стремление создавать _красивый_ код, умение доводить задачи до конца, умение самостоятельно искать информацию.

## **Навыки**
- язык разработки ABAP (ALV-Grid, ALV-Tree, разработка транзакций, отчетов, интерфейса пользователя, оптимизация кода и т.д.);
- в рамках системы SAP ERP:
    - опыт использования User-Exit, BAdi, Enhancements, BAPI, RFC, разработка oData и REST веб-сервисов;
    - формирование отчетов в формате MS Office (ZWWW, XLSX Workbench);
    - создание pdf-формуляров (Adobe LiveCycle Designer);
    - интеграция с внешними системами с помощью SAP PI/XI, SAP .NET Connector, применение технологии Single Sign-On (SSO);
    - опыт работы с модулями RE-FX, HR, RCM;
- базовые знания SQL и PL/SQL (СУБД Oracle);
- начальные знания языков программирования C++, Java, VBA.

## **Пример кода (ABAP)**
```
CLASS zcf_reca_dh4858_excel_parser DEFINITION
  PUBLIC
  FINAL
  CREATE PUBLIC.

  PUBLIC SECTION.
    CONSTANTS:
      mc_eng_letters_count TYPE i VALUE 26.

    CLASS-METHODS class_constructor.

    CLASS-METHODS create
      IMPORTING iv_file_name       TYPE string
      RETURNING VALUE(ro_instance) TYPE REF TO zif_reca_dh4858_excel_parser
      RAISING zcx_dh4858_reca.

    CLASS-METHODS check_file_name
      IMPORTING iv_file_name TYPE string
      RAISING zcx_dh4858_reca.

    CLASS-METHODS split_file_name
      IMPORTING iv_file_name     TYPE string
      EXPORTING ev_only_path     TYPE string
                ev_name_with_ext TYPE string
                ev_only_name     TYPE string
                ev_only_ext      TYPE string.

  PRIVATE SECTION.
    CLASS-DATA st_abcde TYPE SORTED TABLE OF char1
                          WITH UNIQUE KEY table_line.
ENDCLASS.


CLASS zcf_reca_dh4858_excel_parser IMPLEMENTATION.

  METHOD class_constructor.

    IF ( st_abcde IS INITIAL ).
      DATA(lv_index) = VALUE i( ).

      DO mc_eng_letters_count TIMES.
        APPEND sy-abcde+lv_index(1) TO st_abcde.
        lv_index = lv_index + 1.
      ENDDO.
    ENDIF.

  ENDMETHOD.


  METHOD create.

    CONSTANTS: lc_xlsx_extension TYPE string VALUE 'XLSX'.

    check_file_name( iv_file_name ).

    split_file_name( EXPORTING iv_file_name = iv_file_name
                     IMPORTING ev_only_ext  = DATA(lv_only_ext) ).

    IF ( to_upper( lv_only_ext ) = lc_xlsx_extension ).
      ro_instance = NEW zcl_reca_dh4858_excel_parser_x( ).
    ELSE.
      ro_instance = NEW zcl_reca_dh4858_excel_parser( ).
    ENDIF.

    ro_instance->init( iv_file_name ).

  ENDMETHOD.

  
  METHOD check_file_name.

    cl_gui_frontend_services=>file_exist( EXPORTING  file                 = iv_file_name
                                          RECEIVING  result               = DATA(lf_result)
                                          EXCEPTIONS cntl_error           = 1
                                                     error_no_gui         = 2
                                                     wrong_parameter      = 3
                                                     not_supported_by_gui = 4
                                                     OTHERS               = 5 ).

    IF ( sy-subrc <> 0 ) OR ( lf_result <> abap_true ).
      MESSAGE e003 WITH iv_file_name INTO DATA(lv_msg) ##NEEDED.
      zcx_dh4858_reca=>raise_from_symsg( ).
    ENDIF.

  ENDMETHOD.


  METHOD split_file_name.

    CLEAR: ev_only_path,
           ev_name_with_ext,
           ev_only_name,
           ev_only_ext.

    TRY.
        cl_bcs_utilities=>split_path( EXPORTING iv_path = iv_file_name
                                      IMPORTING ev_path = ev_only_path
                                                ev_name = ev_name_with_ext ).
      CATCH cx_bcs.
        RETURN.
    ENDTRY.

    cl_bcs_utilities=>split_name( EXPORTING iv_name      = ev_name_with_ext
                                  IMPORTING ev_name      = ev_only_name
                                            ev_extension = ev_only_ext ).

  ENDMETHOD.

ENDCLASS.
```
