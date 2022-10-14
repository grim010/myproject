import rclpy
from rclpy.node import Node
import getch #theoretically gets keyboard input. need pip3 to install
#to install 'pip3 install getch'

from geometry_msgs.msg import Twist #import geometry stuff
from std_msgs.msg import String #for pushing info to terminal

msg = """
Taking input from keyboard and publishing to /cmd_vel
Movement:
q w e up/l up up/r
  a d left right
    s down
    r accelerate by 10%
    t deccelerate by 10%
Hit esc to exit
-----------------
"""

def main(args=None):
    rclpy.init(args=args)

    node = rclpy.create_node('keyboard_input3')
    publisher = node.create_publisher(Twist, '/cupcar0/cmd_vel', 10)

    moveBindings = {
    'w': (1, 0, 0, 0),
    'e': (1, 0, 0, -1),
    'd': (0, 0, 0, 1),
    'a': (0, 0, 0, -1),
    'q': (1, 0, 0, 1),
    's': (-1, 0, 0, 0),
    'z': (-1, 0, 0, 1),
    'c': (-1, 0, 0, -1),   
    
    }

    speedBindings = {
    'r': (1.1, 1.1),
    't': (.9, .9),
    }

    def get_keys(): #gets keyboard input
        k = getch.getch() #converts keypress to ord value
        return k

    def velocityOut(speed,turn): #for printing/getting speed & turn angle
        return "speed & turn angle: \tspeed %s\tturn %s " % (speed,turn)


    def keyboard_input():
        speed = 0.5 #default speed val
        turn = 1.0 #default turn val
        x = 0
        #z = 0
        th = 0
	
        while (1):
            input = get_keys()
            if input in moveBindings.keys():
                x = moveBindings[input][0]
                #z = moveBindings[input][2]
                th = moveBindings[input][3]
            elif input in speedBindings.keys():
                speed = speed * speedBindings[input][0]
                turn = turn * speedBindings[input][1]
                print(velocityOut(speed, turn))
            else:
                x = 0
                #z = 0
                th = 0

            print(input)
            twist = Twist()
            twist.linear.x = x*speed
            twist.angular.x = 0.0
            twist.angular.z = th*turn
            publisher.publish(twist)
            print(velocityOut(speed,twist.angular.z)) #write speed/angle to terminal

    print(msg)
    keyboard_input()
    rclpy.spin(node)


    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
