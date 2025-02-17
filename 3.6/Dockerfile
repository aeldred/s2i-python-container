# This image provides a Python 3.6 environment you can use to run your Python
# applications.
FROM centos/s2i-base-centos7

EXPOSE 8080

ENV PYTHON_VERSION=3.6 \
    PATH=$HOME/.local/bin/:$PATH \
    PYTHONUNBUFFERED=1 \
    PYTHONIOENCODING=UTF-8 \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    PIP_NO_CACHE_DIR=off


ENV SUMMARY="Platform for building and running Python $PYTHON_VERSION applications" \
    DESCRIPTION="Python $PYTHON_VERSION available as container is a base platform for \
building and running various Python $PYTHON_VERSION applications and frameworks. \
Python is an easy to learn, powerful programming language. It has efficient high-level \
data structures and a simple but effective approach to object-oriented programming. \
Python's elegant syntax and dynamic typing, together with its interpreted nature, \
make it an ideal language for scripting and rapid application development in many areas \
on most platforms."

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="Python 3.6" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,python,python36,python-36,rh-python36" \
      com.redhat.component="python36-container" \
      name="centos/python-36-centos7" \
      version="1" \
      usage="s2i build https://github.com/sclorg/s2i-python-container.git --context-dir=3.6/test/setup-test-app/ centos/python-36-centos7 python-sample-app" \
      maintainer="SoftwareCollections.org <sclorg@redhat.com>"

ADD oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm /tmp/
ADD oracle-instantclient12.2-devel-12.2.0.1.0-1.x86_64.rpm /tmp/

RUN INSTALL_PKGS="rh-python36 rh-python36-python-devel rh-python36-python-setuptools rh-python36-python-pip nss_wrapper \
        httpd24 httpd24-httpd-devel httpd24-mod_ssl httpd24-mod_auth_kerb httpd24-mod_ldap \
        httpd24-mod_session atlas-devel gcc-gfortran libffi-devel libtool-ltdl enchant wget libaio-devel" && \
    yum install -y centos-release-scl && \
    yum -y install /tmp/oracle-instantclient*.rpm && \
    yum -y --setopt=tsflags=nodocs install --enablerepo=centosplus $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    # Remove centos-logos (httpd dependency) to keep image size smaller.
    rpm -e --nodeps centos-logos && \
    rm -f /tmp/oracle-instantclient*.rpm && \
    yum -y clean all --enablerepo='*' && \
    echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient12.2.conf && \
    ldconfig

# install the Oracle dependencies./tmp/oracle_fdw-ORACLE_FDW_2_0_0/oracle_fdw.control
#COPY oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm /tmp/oraclelibs/oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm
#COPY oracle-instantclient12.2-devel-12.2.0.1.0-1.x86_64.rpm /tmp/oraclelibs/oracle-instantclient12.2-devel-12.2.0.1.0-1.x86_64.rpm

#RUN cd /tmp/oraclelibs && \
#    rpm -Uvh oracle-instantclient12.2-basic-12.2.0.1.0-1.x86_64.rpm && \
#    rpm -Uvh oracle-instantclient12.2-devel-12.2.0.1.0-1.x86_64.rpm

# set oracle env vars
ENV ORACLE_HOME /usr/lib/oracle/12.2/client64/lib
ENV LD_LIBRARY_PATH /usr/lib/oracle/12.2/client64/lib:${LD_LIBRARY_PATH}
ENV PATH=$PATH:/usr/lib/oracle/12.2/client64/bin

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH.
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Copy extra files to the image.
COPY ./root/ /

# - Create a Python virtual environment for use by any application to avoid
#   potential conflicts with Python packages preinstalled in the main Python
#   installation.
# - In order to drop the root user, we have to make some directories world
#   writable as OpenShift default security model is to run the container
#   under random UID.
RUN source scl_source enable rh-python36 && \
    virtualenv ${APP_ROOT} && \
    chown -R 1001:0 ${APP_ROOT} && \
    fix-permissions ${APP_ROOT} -P && \
    rpm-file-permissions

USER 1001

# Set the default CMD to print the usage of the language image.
CMD $STI_SCRIPTS_PATH/usage
