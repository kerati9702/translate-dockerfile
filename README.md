### ENTRYPOINT
การใช้งาน ```bashENTRYPOINT``` ที่ดีที่สุดคือการตั้งค่าคำสั่งหลักของimageเพื่อให้สามารถเรียกใช้imageเหมือนเป็นคำสั่งนั้น (จากนั้นใช้ CMD เป็นค่าเริ่มต้น)

เริ่มด้วยตัวอย่างของimageของcommand line tool ```bashs3cmd```:

```bash
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```

โดยimageสามารถทำงานได้เช่นนี้เพื่อแสดงความช่วยเหลือของคำสั่ง:

```bash
$ docker run s3cmd
```

หรือใช้พารามิเตอร์ที่เหมาะสมเพื่อใช้คำสั่ง:

```bash
$ docker run s3cmd ls s3://mybucket
```

วิธีนี้มีประโยชน์อย่างมากเพราะชื่อของimageจะเพิ่มเป็นสองเท่า ดังที่แสดงในคำสั่งด้านบน
คำสั่ง ```bashENTRYPOINT``` ยังสามารถใช้ร่วมกับตัวช่วยสคริปต์ ซึ่งสามารถทำงานได้ในลักษณะเดียวกันกับคำสั่งด้านบน ถึงแม้ว่าช่วงเริ่มอาจจะใช้มากกว่าหนึ่งขั้นตอน
ตัวอย่างเช่น imageอย่างเป็นทางการของ Postgres ใช้สคริปต์ต่อไปนี้เป็น ```bashENTRYPOINT```

```bash
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

ตัวช่วยสคริปต์จะถูกคัดลอกลงในคอนเทนเนอร์และถูกเรียกใช้ผ่าน ENTRYPOINT เมื่อเริ่มต้นคอนเทนเนอร์:

```bash
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["postgres"]
```


สคริปต์นี้อนุญาตให้ผู้ใช้โต้ตอบกับ Postgres ได้หลายวิธี
โดยสามารถเริ่ม Postgres จาก :

```bash
$ docker run postgres
```

หรือสามารถใช้เพื่อรัน Postgres และส่งต่อพารามิเตอร์ไปยังเซิร์ฟเวอร์:

```bash
$ docker run postgres postgres --help
```


และสุดท้ายก็สามารถใช้เพื่อเริ่มเครื่องมือที่แตกต่างอย่างสิ้นเชิงเช่น Bash:

```bash
$ docker run --rm -it postgres bash
```

### VOLUME  //Short
การใช้คำสั่ง ```bashVOLUME``` จะใช้เพื่อแสดงพื้นที่ของฐานข้อมูล,การจัดเก็บการกำหนดค่า,ไฟล์ หรือโฟลเดอร์ที่สร้างโดยdocker container โดยควรใช้VOLUMEสำหรับส่วนของimageที่เปลี่ยนแปลงง่าย
และ/หรือส่วนที่userไม่สามารถแก้ไขได้
### USER
ถ้า service นั้นสามารถใช้ได้แบบไม่มีสิทธิพิเศษ(privilage) จะสามารถใช้ ```bashUSER``` เพื่อเปลี่ยนเป็นผู้ใช้แบบ non-root โดยเริ่มต้นด้วยการสร้าง```bashUSER```และจัดกลุ่ม Dockerfile อย่างเช่น

```bash
RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres.
```
หลีกเลี่ยงการติดตั้งหรือการใช้ sudo มีลักษณะการทำงานของ TTY ที่คาดเดาไม่ได้และลักษณะการส่งต่อซึ่งอาจทำให้เกิดปัญหาได้ หากคุณต้องการฟังก์ชั่นที่คล้ายกับ sudo เช่นการเริ่มต้น daemon เป็นรูท แต่รันเป็น non-root ให้ลองใช้ “gosu”
สุดท้ายเพื่อลดความซับซ้อน ให้หลีกเลี่ยงการสลับ USER ไปมาบ่อยๆ
### WORKDIR //Short
เพื่อความชัดเจนและความน่าเชื่อถือ ควรใช้เส้นทางที่แน่นอนสำหรับ WORKDIR  นอกจากนี้ควรใช้ WORKDIR แทนคำแนะนำที่เพิ่มขึ้นเช่น  ```bash RUN cd ... &&- ```ซึ่งยากต่อการอ่าน,แก้ไขและบำรุงรักษา
### ONBUILD 
คำสั่ง ```bashONBUILD``` จะดำเนินการหลังจากการสร้าง Dockerfile ปัจจุบันเสร็จ ```bashONBUILD``` จะดำเนินการในchild imageใด ๆ ที่ได้รับจาก image ปัจจุบัน โดย Docker Build จะเรียกใช้งานคำสั่ง ```bashONBUILD``` ก่อนคำสั่งใด ๆ ใน child  ```bashONBUILD``` มีประโยชน์สำหรับimageที่กำลังจะสร้างจากimageที่กำหนด ตัวอย่างเช่นคุณจะใช้```bash ONBUILD``` สำหรับimageแบบstackภาษาที่สร้างซอฟแวร์ผู้ใช้ที่เขียนด้วยภาษานั้นภายใน Dockerfile ดังที่คุณเห็นในRuby’s ONBUILD รุ่นต่างๆ 

รูปภาพที่สร้างด้วย```bash ONBUILD ```ควรได้รับTagแยกต่างหากตัวอย่างเช่น:```bash ruby: 1.9-onbuild ```หรือ ```bashruby: 2.0-onbuild```
ควรระวังเมื่อ ```bashADD ```หรือ ```bashCOPY``` ลงใน ```bashONBUILD ```image“ onbuild” ล้มเหลว หากบริบทของบิลด์ใหม่ไม่มีทรัพยากรที่เพิ่มเข้ามาการเพิ่มแท็กแยกตามที่แนะนำไว้ข้างต้นจะช่วยลดสิ่งนี้ได้โดยอนุญาตให้ผู้เขียน ```bashDockerfile``` .
