import tkinter as tk
import time
import sys
from datetime import datetime
from datetime import timedelta
from collections import deque
import requests
from urllib.request import urlopen
import json
import urllib
import os
import threading
from urllib.parse import urlencode, unquote, quote_plus
from PIL import ImageTk, Image

#윈도우 생성
window = tk.Tk()
window.title("smart mirror")
window.configure(bg='black')
window.geometry("800x480")

#날씨 데이터 캔버스 생성
canvas = tk.Canvas(window, width=800, height=150, bg="black", scrollregion=(0, 0, 1200, 150),highlightbackground="black", highlightthickness=1)
canvas.grid(row=8, column=0, sticky="nsew", columnspan=9)  # grid 매니저를 사용하여 배치

#날씨 데이터 스크롤바 생성
scrollbar = tk.Scrollbar(window, orient="horizontal",command=canvas.xview)
canvas.configure(xscrollcommand=scrollbar.set)

#마우스 휠 기능
def horizontal_scroll(event):
    canvas.xview_scroll(-1 * (event.delta // 120), "units")

#데이터를 표시할 프레임 생성
frame = tk.Frame(canvas,bg='black')
canvas.create_window((0, 0), window=frame, anchor="nw")


bus_canvas = tk.Canvas(window, width=100, height=80, bg="black", scrollregion=(0, 0, 100, 200),highlightbackground="black", highlightthickness=1)
bus_canvas.grid(row=3, column=0, columnspan=2,sticky="nsew")  # grid 매니저를 사용하여 배치

bus_scrollbar = tk.Scrollbar(window, orient="vertical", command=bus_canvas.yview)

bus_canvas.configure(yscrollcommand=scrollbar.set)

def vertical_scroll(event):
    # 마우스 휠 이벤트에 대한 세로 스크롤 기능
    bus_canvas.yview_scroll(-1 * (event.delta // 120), "units")

# 가로 스크롤 동작 설정
canvas.bind("<Enter>", lambda event: canvas.bind_all("<MouseWheel>", horizontal_scroll))
canvas.bind("<Leave>", lambda event: canvas.unbind_all("<MouseWheel>"))

bus_canvas.bind("<Enter>", lambda event: bus_canvas.bind_all("<MouseWheel>", vertical_scroll))
bus_canvas.bind("<Leave>", lambda event: bus_canvas.unbind_all("<MouseWheel>"))


# 데이터를 표시할 프레임 생성
bus_frame = tk.Frame(bus_canvas, bg="black")
bus_canvas.create_window((0, 0), window=bus_frame, anchor="nw")


#버스도착정보 텍스트
text = bus_canvas.create_text(0,0,text="",fill="white", font=("Product Sans", 20), anchor="nw")


#버스 도착 정보 키 입력
bus_url='http://apis.data.go.kr/1613000/ArvlInfoInqireService/getSttnAcctoArvlPrearngeInfoList'
bus_serviceKey = "4UZE%2BEtRxVVwIiH7BTz7mjplmcrtjsNue%2BiFD6qZ%2F88QWc5jgcjlu%2BGoNnPywNhO0HKAL1d%2FMRc08e32b4o57w%3D%3D"
bus_serviceKeyDecoded = unquote(bus_serviceKey,'UTF-8')

#버스 도착 정류소
node_Id='CJB283000214'

#날씨 요청키 입력
url= 'http://apis.data.go.kr/1360000/VilageFcstInfoService_2.0/getVilageFcst'


serviceKey = "4UZE%2BEtRxVVwIiH7BTz7mjplmcrtjsNue%2BiFD6qZ%2F88QWc5jgcjlu%2BGoNnPywNhO0HKAL1d%2FMRc08e32b4o57w%3D%3D"
serviceKeyDecoded = unquote(serviceKey,'UTF-8')
#청주시 위치정보
nx=68
ny=106
#날씨데이터 자료저장변수
weather_data = dict()
#업데이트 대비 자료저장 변수
next_weather_data= dict()

user_index=0
user_list=['이현욱']
shared_path_up = None
shared_path_down = None
def button_clicked():
    global current_index
    global shared_path_up
    global image_folders
    current_index += 1
    if current_index >= len(shared_path_up):
        current_index = 0
    image_file = Image.open(shared_path_up[current_index])
    size=(100,100)
    resized_image=image_file.resize(size)
    image = ImageTk.PhotoImage(resized_image)
   
    image_label.config(image=image)
    image_label.image = image


def button_clicked_2():
    global current_index_2
    global image_folders
    global shared_path_down
    current_index_2 += 1
    if current_index_2 >= len(shared_path_down):
        current_index_2 = 0
    image_file_2 = Image.open(shared_path_down[current_index_2])
    size=(100,100)
    resized_image_2=image_file_2.resize(size)
    image_2 = ImageTk.PhotoImage(resized_image_2)
   
    image_label_2.config(image=image_2)
    image_label_2.image = image_2


def button_clicked_4():
    global user_index
    global shared_path_up
    global shared_path_down
   
   
    # 다음 인덱스로 업데이트
    user_index = (user_index+ 1) % len(user_list)
    user_label.config(text=user_list[user_index])
    image_folders=[
        r"/home/pi/Desktop" + "/" + user_list[user_index] + '/' + "어른을 만나는 자리",
        r"/home/pi/Desktop" + "/" + user_list[user_index] + '/' + "친구를 만나는 자리",
        r"/home/pi/Desktop" + "/" + user_list[user_index] + '/' + "결혼식장",
        r"/home/pi/Desktop" + "/" + user_list[user_index] + '/' + "장례식장",
        r"/home/pi/Desktop" + "/" + user_list[user_index] + '/' + "소개팅"
    ]
    image_paths=image_folders[current_index_3]
    image_path_up=image_paths+"/상의"
    image_path_down=image_paths+"/하의"
   
   
    image_path_up_list=[os.path.join(image_path_up, f) for f in os.listdir(image_path_up) if f.endswith('.png') or f.endswith('.jpg') or f.endswith('.jfif')]
    image_path_down_list=[os.path.join(image_path_down, f) for f in os.listdir(image_path_down) if f.endswith('.png') or f.endswith('.jpg') or f.endswith('.jfif')]
    shared_path_up = image_path_up_list
    shared_path_down = image_path_down_list
    # Create an image object from file
    image_file = Image.open(image_path_up_list[0])
    size=(100,100)
    resized_image=image_file.resize(size)
    image = ImageTk.PhotoImage(resized_image)
    image_label.config(image=image)
    image_label.image = image
    # Create an image object from file
    image_file_2 = Image.open(image_path_down_list[0])
    resized_image_2=image_file_2.resize(size)
    image_2 = ImageTk.PhotoImage(resized_image_2)
    image_label_2.config(image=image_2)
    image_label_2.image = image_2

#"C:\Users\82103\Desktop\이현욱\결혼식장\상의\1.jpg"
situation_list=['어른을 만나는 자리','친구를 만나는 자리','결혼식장','장례식장','소개팅']
image_folders=[
        r"/home/pi/Desktop" + "/" + user_list[user_index] + '/' + "어른을 만나는 자리",
        r"/home/pi/Desktop" + "/" + user_list[user_index] + '/' + "친구를 만나는 자리",
        r"/home/pi/Desktop" + "/" + user_list[user_index] + '/' + "결혼식장",
        r"/home/pi/Desktop" + "/" + user_list[user_index] + '/' + "장례식장",
        r"/home/pi/Desktop" + "/" + user_list[user_index] + '/' + "소개팅"
    ]
current_index_3 = 0
def button_clicked_3():
    global current_index_3
    global image_folders
    global shared_path_up
    global shared_path_down
   
    current_index_3 = (current_index_3 + 1) % len(situation_list)
    button_3.config(text=situation_list[current_index_3])
    image_paths=image_folders[current_index_3]
    image_path_up=image_paths+"/상의"
    image_path_down=image_paths+"/하의"

    image_path_up_list=[os.path.join(image_path_up, f) for f in os.listdir(image_path_up) if f.endswith('.png') or f.endswith('.jpg') or f.endswith('.jfif')]
    image_path_down_list=[os.path.join(image_path_down, f) for f in os.listdir(image_path_down) if f.endswith('.png') or f.endswith('.jpg') or f.endswith('.jfif')]
    shared_path_up = image_path_up_list
    shared_path_down = image_path_down_list
    # Create an image object from file
    image_file = Image.open(image_path_up_list[0])
    size=(100,100)
    resized_image=image_file.resize(size)
    image = ImageTk.PhotoImage(resized_image)
    image_label.config(image=image)
    image_label.image = image
    # Create an image object from file
    image_file_2 = Image.open(image_path_down_list[0])
    resized_image_2=image_file_2.resize(size)
    image_2 = ImageTk.PhotoImage(resized_image_2)
    image_label_2.config(image=image_2)
    image_label_2.image = image_2

def button_clicked_ok():
   
    """gpio_thread=threading.Thread(target=run_gpio)"""
    """gpio_thread.start()"""
    # Hide all other screens
    button.grid_remove()
    button_2.grid_remove()
    button_3.grid_remove()
    image_label.grid_remove()
    image_label_2.grid_remove()
    button_ok.grid_remove()
    user_label.grid_remove()
   

    # Display "약속 잘 다녀오세요" in the center
    goodbye_label = tk.Label(window, text="약속 잘 다녀오세요", font=("Product Sans", 20),fg="white", bg="black")
    goodbye_label.grid(row=2, column=2, columnspan=7, padx=10, pady=10)
   
def button_reset():
    global current_index, current_index_2, current_index_3
   
    # 초기 인덱스로 변경
    current_index = 0
    current_index_2 = 0
    current_index_3 = 0
   
    # 이미지 초기화
    image_file = Image.open(shared_path_up[current_index])
    resized_image = image_file.resize((100, 100))
    image = ImageTk.PhotoImage(resized_image)
    image_label.config(image=image)
    image_label.image = image

    image_file_2 = Image.open(shared_path_down[current_index_2])
    resized_image_2 = image_file_2.resize((100, 100))
    image_2 = ImageTk.PhotoImage(resized_image_2)
    image_label_2.config(image=image_2)
    image_label_2.image = image_2

    # 상황 선택 초기화
    button_3.config(text=situation_list[current_index_3])
   
    # 프로그램 종료
    window.destroy()
    sys.exit()
"""def run_gpio():
    import RPi.GPIO as GPIO
    import time

    led_pin = 17
    led_pin_1 = 27
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(led_pin, GPIO.OUT)
    GPIO.setup(led_pin_1, GPIO.OUT)

    # Create a PWM object with frequency 100 Hz and 100% duty cycle
    pwm = GPIO.PWM(led_pin, 100)
    pwm.start(100)
    pwm_1 = GPIO.PWM(led_pin_1, 100)
    pwm_1.start(100)
   
    def turn_off_led():
     time.sleep(5)
     pwm.ChangeDutyCycle(0)
     pwm_1.ChangeDutyCycle(0)
     sys.exit()
    try:
        # Turn on the LED for 5 seconds
        pwm.ChangeDutyCycle(100)
        pwm_1.ChangeDutyCycle(100)
        t=threading.Thread(target=turn_off_led)
        t.start()
       
        while True:
            time.sleep(1)

    except KeyboardInterrupt:
    # Clean up GPIO settings on keyboard interrupt
     pwm.stop()
     pwm_1.stop()
     GPIO.cleanup()
"""
# Set the path to the image folder
image_paths=image_folders[current_index_3]
image_path_up=image_paths+"/상의"
image_path_down=image_paths+"/하의"

image_path_up_list=[os.path.join(image_path_up, f) for f in os.listdir(image_path_up) if f.endswith('.png') or f.endswith('.jpg') or f.endswith('.jfif')]
image_path_down_list=[os.path.join(image_path_down, f) for f in os.listdir(image_path_down) if f.endswith('.png') or f.endswith('.jpg') or f.endswith('.jfif')]

# Create an image object from file
image_file = Image.open(image_path_up_list[0])
size=(100,100)
resized_image=image_file.resize(size)
image = ImageTk.PhotoImage(resized_image)

# Create an image object from file
image_file_2 = Image.open(image_path_down_list[0])
resized_image_2=image_file_2.resize(size)
image_2 = ImageTk.PhotoImage(resized_image_2)

# Create a label for the image
image_label = tk.Label(window, image=image)
image_label.grid(row=0, column=1, padx=10, pady=1, rowspan=4, columnspan=3, sticky="w")


# Create a label for the image
image_label_2 = tk.Label(window, image=image_2)
image_label_2.grid(row=3, column=1, padx=10, pady=1, rowspan=4, columnspan=3, sticky="w")

# Create a button1
button = tk.Button(window, text='다음 이미지', command=button_clicked)
button.grid(row=3, column=4, padx=10, pady=10)

# Create a button1
button_2 = tk.Button(window, text='다음 이미지', command=button_clicked_2)
button_2.grid(row=5, column=4, padx=10, pady=10)

# Create a button(diary)
button_3 = tk.Button(window, text=situation_list[current_index_3], command=button_clicked_3)
button_3.grid(row=0, column=1, padx=10, pady=10, sticky="w")

# Create the OK button
button_ok = tk.Button(window, text="OK", command=button_clicked_ok)
button_ok.grid(row=5, column=5, padx=10, pady=10)

# Create the reset button
button_reset = tk.Button(window, text="초기화", command=button_clicked_ok)
button_reset.grid(row=0, column=5, padx=10, pady=10)

#버튼 속성 변경
button.configure(bg="black", fg="white", font=("Product Sans", 14))
#버튼 속성 변경
button_2.configure(bg="black", fg="white", font=("Product Sans", 14))
#버튼 속성 변경
button_3.configure(bg="black", fg="white", font=("Product Sans", 14))
#버튼 속성 변경
button_ok.configure(bg="black", fg="white", font=("Product Sans", 14))
#버튼 속성 변경
button_reset.configure(bg="black", fg="white", font=("Product Sans", 14))



# Initialize the current index to 0
current_index = 0
current_index_2 = 0
#버스도착정보 리스트
BusList=[[None,None] for _ in range(1)]

#사용자 이름
user_label =tk.Label(window, font=("Product Sans", 20), text=user_list[0],fg="white", bg="black")
user_label.grid(row=0, column=4, padx=10, pady=1, sticky="w")

#현재날짜 라벨 만들기
today_label = tk.Label(window, font=("Product Sans", 20), fg="white", bg="black")
today_label.grid(row=0, column=0, padx=10, pady=1, sticky="w")
# 현재 시간 라벨 만들기
time_label = tk.Label(window, font=("Product Sans", 16),fg="white", bg="black")
time_label.grid(row=1, column=0, padx=10, pady=1, sticky="w")

#버스 도착정보 라벨 만들기
bus_inform_name = tk.Label(window,font=("Product Sans", 20),fg="white", bg="black")
bus_inform_name.grid(row=2,column=0, padx=10, pady=10, sticky="w")


#현재 위치 라벨 만들기
location_label = tk.Label(window, font=("Product Sans", 24),fg="white", bg="black")
location_label.grid(row=5, column=0, padx=0, pady=0, sticky="w")

# 아이콘 파일 경로 설정
icon_path_sunny = r"/home/pi/Downloads/맑음.png"
icon_path_little_cloudy = r"/home/pi/Downloads/조금흐림.png"
icon_path_cloudy=r"/home/pi/Downloads/흐림.png"
icon_path_rain=r"/home/pi/Downloads/맑음.png"
# 이미지 불러오기
icon_sunny = tk.PhotoImage(file=icon_path_sunny)
icon_little_cloudy = tk.PhotoImage(file=icon_path_little_cloudy)
icon_cloudy = tk.PhotoImage(file=icon_path_cloudy)
icon_rain = tk.PhotoImage(file=icon_path_rain)


# 이미지 크기 조정
font_size = 50
icon_sunny = icon_sunny.subsample(icon_sunny.width() // font_size, icon_sunny.height() // font_size)
icon_little_cloudy = icon_little_cloudy.subsample(icon_little_cloudy.width() // font_size, icon_little_cloudy.height() // font_size)
icon_cloudy = icon_cloudy.subsample(icon_cloudy.width() // font_size, icon_cloudy.height() // font_size)
icon_rain = icon_rain.subsample(icon_rain.width() // font_size, icon_rain.height() // font_size)


           
#이전 base_time저장
pre_base_time='?'
#base_time저장
base_time=''

#이전 minute저장
pre_minute=-1

def update_time():
    #시간가져오기
    now=datetime.now()
    #현재날짜 변수
    today_str = time.strftime("%Y년%m월%d일", time.localtime())
    #현재시간 변수
    clock = time.strftime("%H:%M")
    #전역변수 사용 선언
    global pre_base_time
    global base_time
    global clothe_index
    global pre_minute
    global BusList
    #base_time에 필요한 변수
    current_time=int(time.strftime("%H%M"))
    #base_date에 필요한 변수
    today = datetime.today().strftime("%Y%m%d")
    y = datetime.today() - timedelta(days=1)
    yesterday=y.strftime("%Y%m%d")
   
   
   

    #요청시간 정하기
    if (current_time>=210 and current_time<510):
      base_time = "0200"
      base_date = today
    elif (current_time>=510 and current_time<810):
      base_time = "0500"
      base_date = today
    elif (current_time>=810 and current_time<1110):
      base_time = "0800"
      base_date = today
    elif (current_time>=1110 and current_time<1410):
      base_time = "1100"
      base_date = today
    elif (current_time>=1410 and current_time<1710):
      base_time = "1400"
      base_date = today
    elif (current_time>=1710 and current_time<2010):
      base_time = "1700"
      base_date = today
    elif (current_time>=2010 and current_time<2310):
      base_time = "2000"
      base_date = today
    elif (current_time>=2310):
      base_time = "2300"
      base_date = today
    else:
      base_time = "2300"
      base_date = yesterday
   
    List_count=0
   
    #1분 지나면 데이터 버스 도착정보 요청
   
    if pre_minute!=now.minute:
      bus_queryParams = '?' + urlencode({ quote_plus('serviceKey') : bus_serviceKeyDecoded,quote_plus('cityCode') : '33010',quote_plus('nodeId') : node_Id,quote_plus('_type') : 'json'})
      res_bus = requests.get(bus_url+bus_queryParams, verify=False)
      items_bus = res_bus.json().get('response').get('body').get('items')
     
      if type(items_bus)!=str:
        for item in items_bus['item']:
          if not 'routeno' in item:
            break
          if List_count==10:
            break
          if List_count==0:
            BusList[0][0]=item['routeno']
            BusList[0][1]=item['arrtime']
          BusList.append([item['routeno'],item['arrtime']])
          List_count+=1
     
    sorted_bus_list = sorted(BusList,key=lambda x:x[1])
   
    pre_minute=now.minute
   
    #업데이트 될시 날씨 자료 업데이트요청
    if pre_base_time!= base_time:
      queryParams = '?' + urlencode({ quote_plus('serviceKey') : serviceKeyDecoded, quote_plus('base_date'): base_date, quote_plus('base_time'):base_time,quote_plus('nx'):nx,quote_plus('ny'):ny,quote_plus('dataType'):'json',quote_plus('numOfRows'):'266'})
      res = requests.get(url + queryParams, verify=False)
      items = res.json().get('response').get('body').get('items')
      #현재시각 기준으로 10개의 데이터를 weather_data에 dict 형식으로 저장하기
      for item in items['item']:
        for i in range(9):
          call_time="{:02d}".format((now.hour+i)%24)
          if item['fcstTime']==call_time+'00':
            if i==1:
              if item['category']=='TMP':
                next_weather_data[item['fcstTime']+'시 기온']=item['fcstValue']
              if item['category']=='SKY':
                next_weather_data[item['fcstTime']+'시 날씨']=item['fcstValue']
              if item['category']=='PCP':
                next_weather_data[item['fcstTime']+'시 강수량']=item['fcstValue']
               
            if item['category']=='TMP':
              weather_data[item['fcstTime']+'시 기온']=item['fcstValue']
            if item['category']=='SKY':
              weather_data[item['fcstTime']+'시 날씨']=item['fcstValue']
            if item['category']=='PCP':
              weather_data[item['fcstTime']+'시 강수량']=item['fcstValue']
    pre_base_time=base_time
   

   
    #각 시간별 온도 표시라벨
    for key in weather_data:
      for i in range(9):
        call_time="{:02d}".format((now.hour+i)%24)
        if key==call_time+'00시 기온':
          if weather_data.get(key,None) is None:
            if i==0:
              temp_label=tk.Label(frame,text=next_weather_data[key]+'℃', font=("Product Sans", 20),fg="white", bg="black")
            else:
              temp_label=tk.Label(frame,text=next_weather_data[key]+'℃', font=("Product Sans", 16),fg="white", bg="black")
            temp_label.grid(row=3, column=i, padx=25, pady=10)
            continue
          if i==0:
            temp_label=tk.Label(frame,text=weather_data[key]+'℃', font=("Product Sans", 20),fg="white", bg="black")
          else:
            temp_label=tk.Label(frame,text=weather_data[key]+'℃', font=("Product Sans", 16),fg="white", bg="black")
          temp_label.grid(row=3, column=i, padx=25, pady=10)
         
          if key=='0000시 기온':
            key='00시 기온'
          else:
            key=key.replace('00','')
          if i==0:
            temp_time_label=tk.Label(frame, text=key,font=("Product Sans", 20),fg="white", bg="black")
          else:
            temp_time_label=tk.Label(frame, text=key,font=("Product Sans", 12),fg="white", bg="black")  
          temp_time_label.grid(row=0,column=i,padx=25,pady=0,rowspan=1)
     
    #각 시간별 아이콘 표시 라벨
    for key in weather_data:
      for i in range(9):
        call_time="{:02d}".format((now.hour+i)%24)
        if key==call_time+'00시 기온':
          if weather_data.get(key,None) is None:
            if next_weather_data[key] != '강수없음':
              icon_label_rain = tk.Label(frame, image=icon_rain,bg="black")
              icon_label_rain.grid(row=2, column=i,padx=25)
              continue
          if weather_data[key] != '강수없음':
            icon_label_rain = tk.Label(frame, image=icon_rain,bg="black")
            icon_label_rain.grid(row=2, column=i,padx=25)

    for key in weather_data:
      for i in range(10):
        call_time="{:02d}".format((now.hour+i)%24)
        if key==call_time+'00시 날씨':
         
          if weather_data.get(key,None) is None:
            if next_weather_data[key]=='1':
              icon_label_sunny = tk.Label(frame, image=icon_sunny,bg="black")
              icon_label_sunny.grid(row=2, column=i,padx=25)
            elif next_weather_data[key]=='3':
              icon_label_little_cloudy = tk.Label(frame, image=icon_little_cloudy,bg="black")
              icon_label_little_cloudy.grid(row=2, column=i,padx=25)
            elif next_weather_data[key]=='4':
              icon_label_cloudy = tk.Label(frame, image=icon_cloudy,bg="black")
              icon_label_cloudy.grid(row=2, column=i,padx=25)
            continue

          if weather_data[key]=='1':
            icon_label_sunny = tk.Label(frame, image=icon_sunny,bg="black")
            icon_label_sunny.grid(row=2, column=i,padx=25)
          elif weather_data[key]=='3':
            icon_label_little_cloudy = tk.Label(frame, image=icon_little_cloudy,bg="black")
            icon_label_little_cloudy.grid(row=2, column=i,padx=25)
          elif weather_data[key]=='4':
            icon_label_cloudy = tk.Label(frame, image=icon_cloudy,bg="black")
            icon_label_cloudy.grid(row=2, column=i,padx=25)
   

    #텍스트 업데이트
    today_label.config(text=today_str)
    time_label.config(text=clock)
    location_label.config(text='청주시의'+now.strftime("%A"))
    bus_inform_name.config(text='버스 정류장:사창사거리')
   
    bus_arr_inform=''
    for i in range(len(sorted_bus_list)):
      if sorted_bus_list[0][0]==None:
        break
      bus_arr_inform+=str(sorted_bus_list[i][0])+'번 '+str(sorted_bus_list[i][1]//60)+'분'
      bus_arr_inform+='\n'

      bus_canvas.itemconfigure(text,text=bus_arr_inform)
    window.after(600000, update_time)



update_time()

window.mainloop()
