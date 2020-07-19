### ENTRYPOINT
การใช้งาน ENTRYPOINT ที่ดีที่สุดคือการตั้งค่าคำสั่งหลักของimageเพื่อให้สามารถเรียกใช้imageเหมือนเป็นคำสั่งนั้น (จากนั้นใช้ CMD เป็นค่าเริ่มต้น)

เริ่มด้วยตัวอย่างของimageของcommand line tool s3cmd:

ENTRYPOINT ["s3cmd"]
CMD ["--help"]

โดยimageสามารถทำงานได้เช่นนี้เพื่อแสดงความช่วยเหลือของคำสั่ง:

$ docker run s3cmd


หรือใช้พารามิเตอร์ที่เหมาะสมเพื่อใช้คำสั่ง:

$ docker run s3cmd ls s3://mybucket

วิธีนี้มีประโยชน์อย่างมากเพราะชื่อของimageจะเพิ่มเป็นสองเท่า ดังที่แสดงในคำสั่งด้านบน
คำสั่ง ENTRYPOINT ยังสามารถใช้ร่วมกับตัวช่วยสคริปต์ ซึ่งสามารถทำงานได้ในลักษณะเดียวกันกับคำสั่งด้านบน ถึงแม้ว่าช่วงเริ่มอาจจะใช้มากกว่าหนึ่งขั้นตอน
ตัวอย่างเช่น imageอย่างเป็นทางการของ Postgres ใช้สคริปต์ต่อไปนี้เป็น ENTRYPOINT

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

ตัวช่วยสคริปต์จะถูกคัดลอกลงในคอนเทนเนอร์และถูกเรียกใช้ผ่าน ENTRYPOINT เมื่อเริ่มต้นคอนเทนเนอร์:

COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["postgres"]


สคริปต์นี้อนุญาตให้ผู้ใช้โต้ตอบกับ Postgres ได้หลายวิธี
โดยสามารถเริ่ม Postgres จาก :

$ docker run postgres

หรือสามารถใช้เพื่อรัน Postgres และส่งต่อพารามิเตอร์ไปยังเซิร์ฟเวอร์:

$ docker run postgres postgres --help


และสุดท้ายก็สามารถใช้เพื่อเริ่มเครื่องมือที่แตกต่างอย่างสิ้นเชิงเช่น Bash:

$ docker run --rm -it postgres bash
### VOLUME  //Short
การใช้คำสั่ง VOLUME จะใช้เพื่อแสดงพื้นที่ของฐานข้อมูล,การจัดเก็บการกำหนดค่า,ไฟล์ หรือโฟลเดอร์ที่สร้างโดยdocker container. โดยควรใช้VOLUMEสำหรับส่วนของimageที่เปลี่ยนแปลงง่าย
และ/หรือส่วนที่userไม่สามารถแก้ไขได้
### USER
ถ้า service นั้นสามารถใช้ได้แบบไม่
### WORKDIR //Short
### ONBUILD 
