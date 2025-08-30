# VMware Host Modules Fix for Linux Kernel 6.16.3

<details>
<summary>English Description</summary>

## Overview

This repository contains patched VMware host modules (`vmmon` and `vmnet`) designed to ensure compatibility with **Linux kernel 6.16.3-200.fc42.x86_64**. The provided fix addresses an issue in the `HostIF_SafeRDMSR` function by replacing the incorrect `rdmsrl_safe` call with `rdmsrq_safe`, enabling successful compilation and installation of VMware modules on modern Linux kernels.

## Problem

The original VMware host modules fail to compile or install correctly on Linux kernel 6.16.3 due to an issue in the `HostIF_SafeRDMSR` function, which handles Model-Specific Register (MSR) reads. The function, located in the `vmmon` module, incorrectly used `rdmsrl_safe`, leading to errors or unreliable behavior on Linux.

Here is the original code with the issue:

```c
/*
 *-----------------------------------------------------------------------------
 *
 * HostIF_SafeRDMSR --
 *
 *      Attempt to read a MSR, and handle the exception if the MSR
 *      is unimplemented.
 *
 * Results:
 *      0 if successful, and MSR value is returned via *val.
 *
 *      If the MSR is unimplemented, *val is set to 0, and a
 *      non-zero value is returned: -1 for Win32, -EIO for Linux,
 *      and 1 for MacOS.
 *
 * Side effects:
 *      None
 *
 *-----------------------------------------------------------------------------
 */
int
HostIF_SafeRDMSR(unsigned int msr,   // IN
                 uint64 *val)        // OUT: MSR value
{
   int err;
   u64 v;

   err = rdmsrl_safe(msr, &v);
   *val = (err == 0) ? v : 0;  // Linux corrupts 'v' on error

   return err;
}
```

The use of `rdmsrl_safe` was incorrect for the intended MSR read operation, causing compilation or runtime issues on Linux kernel 6.16.3.

## Solution

The patched version of the VMware host modules in this repository replaces `rdmsrl_safe` with `rdmsrq_safe`, ensuring proper handling of MSR reads. The corrected code is as follows:

```c
int
HostIF_SafeRDMSR(unsigned int msr,   // IN
                 uint64 *val)        // OUT: MSR value
{
  int err;
  u64 v;

  err = rdmsrq_safe(msr, &v);
  *val = (err == 0) ? v : 0;  // Linux corrupts 'v' on error

  return err;
}
```

This fix ensures that the modules compile and install without errors, enabling seamless operation of VMware Workstation Pro on Fedora 42.

## Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/vsemke/vmware-host-modules.git
   cd vmware-host-modules
   ```

2. Build and install the modules:
   ```bash
   make
   sudo make install
   ```

3. Create tar archives for the modules and copy them to the VMware modules directory:
   ```bash
   tar -cf vmmon.tar vmmon-only
   tar -cf vmnet.tar vmnet-only
   sudo cp -v vmmon.tar vmnet.tar /usr/lib/vmware/modules/source/
   ```

4. Configure and install VMware modules:
   ```bash
   sudo vmware-modconfig --console --install-all
   ```

5. Verify that the VMware services are running:
   ```bash
   sudo systemctl restart vmware
   sudo systemctl status vmware
   ```

## Compatibility

- **Linux Kernel**: 6.16.3-200.fc42.x86_64
- **Distribution**: Fedora 42
- **VMware Version**: Tested with VMware Workstation 17 Pro (17.6.3 build-24583834)

## License

The code in this repository is based on VMware's open-source host modules and is distributed under the same license as the original source. See the `LICENSE` file for details.

## Contributing

Feel free to submit issues or pull requests if you encounter problems or have improvements to suggest. Ensure that any contributions are compatible with the target kernel version and follow the existing code style.

</details>

<details>
<summary>Описание на русском</summary>

## Обзор

Этот репозиторий содержит исправленные модули хоста VMware (`vmmon` и `vmnet`), обеспечивающие совместимость с **ядром Linux 6.16.3-200.fc42.x86_64**. Исправление устраняет проблему в функции `HostIF_SafeRDMSR`, заменяя некорректный вызов `rdmsrl_safe` на `rdmsrq_safe`, что позволяет успешно компилировать и устанавливать модули VMware на современных ядрах Linux.

## Проблема

Оригинальные модули хоста VMware не компилируются и не устанавливаются корректно на ядре Linux 6.16.3 из-за ошибки в функции `HostIF_SafeRDMSR`, которая отвечает за чтение регистров MSR (Model-Specific Register). Функция, расположенная в модуле `vmmon`, использовала `rdmsrl_safe`, что приводило к ошибкам компиляции или нестабильному поведению.

Исходный код с проблемой:

```c
/*
 *-----------------------------------------------------------------------------
 *
 * HostIF_SafeRDMSR --
 *
 *      Attempt to read a MSR, and handle the exception if the MSR
 *      is unimplemented.
 *
 * Results:
 *      0 if successful, and MSR value is returned via *val.
 *
 *      If the MSR is unimplemented, *val is set to 0, and a
 *      non-zero value is returned: -1 for Win32, -EIO for Linux,
 *      and 1 for MacOS.
 *
 * Side effects:
 *      None
 *
 *-----------------------------------------------------------------------------
 */
int
HostIF_SafeRDMSR(unsigned int msr,   // IN
                 uint64 *val)        // OUT: MSR value
{
   int err;
   u64 v;

   err = rdmsrl_safe(msr, &v);
   *val = (err == 0) ? v : 0;  // Linux corrupts 'v' on error

   return err;
}
```

Использование `rdmsrl_safe` было некорректным для операции чтения MSR, что вызывало проблемы на ядре Linux 6.16.3.

## Решение

Исправленная версия модулей VMware в этом репозитории заменяет `rdmsrl_safe` на `rdmsrq_safe`, обеспечивая корректную обработку чтения MSR. Исправленный код:

```c
int
HostIF_SafeRDMSR(unsigned int msr,   // IN
                 uint64 *val)        // OUT: MSR value
{
  int err;
  u64 v;

  err = rdmsrq_safe(msr, &v);
  *val = (err == 0) ? v : 0;  // Linux corrupts 'v' on error

  return err;
}
```

Этот фикс гарантирует, что модули компилируются и устанавливаются без ошибок, обеспечивая стабильную работу VMware Workstation Pro на Fedora 42.

## Установка

1. Склонируйте репозиторий:
   ```bash
   git clone https://github.com/vsemke/vmware-host-modules.git
   cd vmware-host-modules
   ```

2. Скомпилируйте и установите модули:
   ```bash
   make
   sudo make install
   ```

3. Создайте архивы для модулей и скопируйте их в директорию VMware:
   ```bash
   tar -cf vmmon.tar vmmon-only
   tar -cf vmnet.tar vmnet-only
   sudo cp -v vmmon.tar vmnet.tar /usr/lib/vmware/modules/source/
   ```

4. Настройте и установите модули VMware:
   ```bash
   sudo vmware-modconfig --console --install-all
   ```

5. Проверьте, что службы VMware запущены:
   ```bash
   sudo systemctl restart vmware
   sudo systemctl status vmware
   ```

## Совместимость

- **Ядро Linux**: 6.16.3-200.fc42.x86_64
- **Д дро**: Fedora 42
- **Версия VMware**: Протестировано с VMware Workstation 17 Pro (17.6.3 build-24583834)

## Лицензия

Код в этом репозитории основан на открытых исходных кодах модулей VMware и распространяется под той же лицензией. Подробности см. в файле `LICENSE`.

## Вклад в проект

Приглашаем сообщать о проблемах или предлагать улучшения через issues или pull requests. Убедитесь, что ваши изменения совместимы с целевой версией ядра и соответствуют стилю кода.

</details>