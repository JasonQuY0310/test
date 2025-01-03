#!/usr/bin/env python
import rospy
from mavros_msgs.msg import State, CommandTOL
from mavros_msgs.srv import SetMode, CommandTOL

def set_mode(mode):
    rospy.wait_for_service('/mavros/set_mode')
    try:
        set_mode_service = rospy.ServiceProxy('/mavros/set_mode', SetMode)
        response = set_mode_service(0, mode)  # 0 is the target system ID, mode is 'GUIDED' or 'HOLD'
        return response.success
    except rospy.ServiceException as e:
        rospy.logerr("Service call failed: %s" % e)
        return False

def takeoff():
    # 发布起飞命令，目标高度2米
    rospy.wait_for_service('/mavros/cmd/arming')
    rospy.wait_for_service('/mavros/cmd/takeoff')
    try:
        arm_service = rospy.ServiceProxy('/mavros/cmd/arming', CommandTOL)
        takeoff_service = rospy.ServiceProxy('/mavros/cmd/takeoff', CommandTOL)

        arm_service(True)  # 解锁飞机
        rospy.loginfo("Arming the vehicle")

        takeoff_service(altitude=2)  # 设置起飞高度2米
        rospy.loginfo("Taking off to 2 meters")

    except rospy.ServiceException as e:
        rospy.logerr("Service call failed: %s" % e)

def hold():
    # 切换到Hold模式
    if set_mode('HOLD'):
        rospy.loginfo("Successfully switched to HOLD mode.")
    else:
        rospy.logerr("Failed to switch to HOLD mode.")

def main():
    rospy.init_node('px4_takeoff_node', anonymous=True)
    rospy.loginfo("PX4 Takeoff Node Started")

    # 等待仿真开始
    rospy.sleep(5)

    takeoff()  # 执行起飞命令
    rospy.sleep(10)  # 等待一会儿让飞机稳定
    hold()  # 切换到Hold模式

    rospy.spin()

if __name__ == "__main__":
    main()
