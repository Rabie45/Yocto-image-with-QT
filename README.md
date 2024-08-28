# Yocto-image-with-QT
Creating a Yocto image with Qt support involves several steps, including configuring the build environment, adding the necessary layers, and customizing the build configuration to include Qt. Below is a step-by-step guide to creating a Yocto image with Qt

## Steps
- In this Demo we will use sato image
- First clone qt layer in yocto path ```git clone -b kirkstone https://github.com/meta-qt5/meta-qt5.git```
- Source poky ```source oe-init-build-env build```
- Add the QT layer ```bitbake-layers add-layer ../meta-qt5```
![Screenshot from 2024-08-28 12-44-40](https://github.com/user-attachments/assets/814d44bc-98e3-43b3-822b-01a3ecc8ffe7)
- Edit local.conf to add recipes for qt
```
IMAGE_INSTALL:append = " qtbase qtdeclarative qtquickcontrols qtbase-tools qtbase-plugins qtexamples"
DISTRO_FEATURES:append = " wayland opengl wifi pam"
CORE_IMAGE_EXTRA_INSTALL += "wayland weston"
IMAGE_FEATURES += " x11-sato x11 "
IMAGE_INSTALL:append = " x11vnc"
IMAGE_INSTALL:append = " mesa"
PREFERRED_PROVIDER_virtual/libgl = "mesa"
```
 - IMAGE_INSTALL:append = " qtbase qtdeclarative qtquickcontrols qtbase-tools qtbase-plugins":

    This appends several Qt-related packages to the list of packages that will be included in your image.
    qtbase: The core Qt module providing basic classes and tools for developing Qt applications.
    qtdeclarative: Provides support for QML and JavaScript-based applications.
    qtquickcontrols: Contains pre-built UI components (buttons, sliders, etc.) for use in QML.
    qtbase-tools: Provides tools related to the Qt base module (e.g., qmake).
    qtbase-plugins: Adds additional plugins to Qt (e.g., platform plugins like EGLFS, XCB, etc.).
    qtexample: for my application name

 - DISTRO_FEATURES:append = " wayland opengl wifi pam":

    wayland: Adds support for the Wayland display server protocol, a modern alternative to X11.
    opengl: Enables OpenGL support, crucial for hardware-accelerated graphics.
    wifi: Ensures Wi-Fi support is included in your image.
    pam: Adds Pluggable Authentication Modules (PAM) for handling authentication tasks.

 - CORE_IMAGE_EXTRA_INSTALL += "wayland weston":

    wayland: Adds the Wayland protocol and libraries.
    weston: Weston is the reference compositor for Wayland. This means that your image will have the capability to run Wayland sessions.

 - IMAGE_FEATURES += " x11-sato x11 ":

    x11-sato: Adds the Sato environment, which is a lightweight graphical environment commonly used in embedded systems.
    x11: Ensures that basic X11 support is included. Even though Wayland is included, X11 support is still available.

 - IMAGE_INSTALL:append = " x11vnc":

    x11vnc: A VNC server that allows remote access to the X11 display. This lets you view and interact with the desktop environment on another machine.

- IMAGE_INSTALL:append = " mesa":

    mesa: An open-source implementation of the OpenGL specification. It provides software rendering support and can also interface with hardware-accelerated graphics drivers.
    Mesa provides the libGL implementation, which is crucial for applications requiring OpenGL.

 - PREFERRED_PROVIDER_virtual/libgl = "mesa":

    This line sets mesa as the preferred provider for the virtual libgl library.
    In Yocto, virtual/libgl is a virtual target that different implementations (such as Mesa or proprietary GPU drivers) can provide.
    By setting mesa as the preferred provider, you're ensuring that the Mesa implementation of OpenGL is used.


## Create QT app
- The Hello World application consists of multiple files  (main.cpp mainwindow.cpp mainwondow.hpp mainwindow.ui main.pro)
- Make a project with QT Creator and make it empty
- Rewrite the ```main.pro`` file to work with yocto
```
  QT       += core gui

greaterThan(QT_MAJOR_VERSION, 4): QT += widgets


TARGET = myqtapp
TEMPLATE = app

SOURCES += \
    main.cpp \
    mainwindow.cpp

HEADERS += \
    mainwindow.h

FORMS += \
    mainwindow.ui

```
##  Lets inject the app in yocto
- Go  to ur cutsom layer if you dont create one
- ``` mkdir recipes-application```
- ``` cd recipes-application```
- ``` mkdir recipes-qt/files```
- ```cd recipes-qt```
  
 ![Screenshot from 2024-08-28 13-00-29](https://github.com/user-attachments/assets/017abb8b-f400-407c-ade9-618c645ab5e5)

- Create application recipe``` touch qtexample_0.1.bb```
- Write the recipe for the application
```
SUMMARY = "Sample Qt5 Application"
DESCRIPTION = "This is a sample Qt5 application."
LICENSE = "CLOSED"


SRC_URI = "file://main.pro \
            file://main.cpp \
            file://mainwindow.cpp \
            file://mainwindow.h \
            file://mainwindow.ui \
            file://ui_mainwindow.h"

DEPENDS += "qtbase wayland"
RDEPENDS_${PN} += "qtwayland"

S = "${WORKDIR}"

inherit qmake5

do_install:append() {
    install -d ${D}${bindir}
    install -m 0775 myqtapp ${D}${bindir}
}
FILES_${PN} += "${bindir}/myqtapp"
```

- But the QT files in files directory
## Build image
- ``` bitbake core-image-sato```
- Then put the image in sdcard and connect
  ![Screenshot from 2024-08-28 13-06-53](https://github.com/user-attachments/assets/9c0db875-5930-4dd8-ba51-a42e5caa18e8)

- press weston 
![Screenshot from 2024-08-28 13-07-31](https://github.com/user-attachments/assets/bd9f7e13-a179-4391-8503-79e4d22b951f)
- write ur app name ```myqtapp``
 ![Screenshot from 2024-08-28 13-10-47-fotor-20240828131622](https://github.com/user-attachments/assets/adcaf229-7ad4-4ce4-9215-b60ae216b4b3)



  
