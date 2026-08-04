FROM brix-ci-base:testing

# 1. groupadd --gid 0 root
# 2. useradd --uid 0 --gid root --shell /bin/sh --home-dir / --system root
# Why: guix pack has no /etc/passwd. Need root user.
RUN groupadd --gid 0 root && \
    useradd --uid 0 --gid root --shell /bin/sh --home-dir / --system root

# 3. cp -rL /gnu/store/*profile/etc/* /etc/
# Why: Copy profile etc files into place.
RUN cp -rL /gnu/store/*profile/etc/* /etc/

# 4. groupadd --system guixbuild
# 5. for i in $(seq -w 1 10); do useradd ... guixbuilder$i; done
# Why: guix-daemon needs 10 build users.
RUN groupadd --system guixbuild && \
    for i in $(seq -w 1 10); do \
        useradd -g guixbuild -G guixbuild -d /var/empty \
            -s "$(command -v nologin)" \
            -c "Guix build user $i" --system guixbuilder$i; \
    done

# 6. export HOME=/
ENV HOME=/

# 7. env LANG=C.UTF-8 guix-daemon --build-users-group=guixbuild &
# Why: Daemon must run before any guix command.
# Wrapped in entrypoint.sh so it starts on every container run.
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["/bin/sh"]

# 8. guix archive --authorize < /share/guix/ci.guix.gnu.org.pub
# 9. guix archive --authorize < /share/guix/bordeaux.guix.gnu.org.pub
# Plus nonguix substitute authorization
RUN guix archive --authorize < /share/guix/ci.guix.gnu.org.pub && \
    guix archive --authorize < /share/guix/bordeaux.guix.gnu.org.pub

# Authorize nonguix channel
RUN wget -q https://substitutes.nonguix.org/signing-key.pub -O /tmp/nonguix.pub && \
    guix archive --authorize < /tmp/nonguix.pub

RUN guix describe
