# project
# Railway Ticket Reservation & Waiting List System
import random
import time
import json
import os
from datetime import datetime



# PASSENGER CLASS


class Passenger:
    def __init__(self, passenger_id, name, age, passenger_type):
        self.passenger_id = passenger_id
        self.name = name
        self.age = age
        self.passenger_type = passenger_type

    def display(self):
        print("Passenger ID   :", self.passenger_id)
        print("Name           :", self.name)
        print("Age            :", self.age)
        print("Passenger Type :", self.passenger_type)



# TICKET CLASS


class Ticket:
    def __init__(self, pnr, passenger, train_no, travel_class,
                 seat_no, fare, status):

        self.pnr = pnr
        self.passenger = passenger
        self.train_no = train_no
        self.travel_class = travel_class
        self.seat_no = seat_no
        self.fare = fare
        self.status = status
        self.booking_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        self.cancellation_time = None

    def display(self):
        print("\n-------------------------------")
        print("PNR              :", self.pnr)
        print("Passenger        :", self.passenger.name)
        print("Age              :", self.passenger.age)
        print("Passenger Type   :", self.passenger.passenger_type)
        print("Train No         :", self.train_no)
        print("Travel Class     :", self.travel_class)
        print("Seat No          :", self.seat_no)
        print("Fare             :", self.fare)
        print("Status           :", self.status)
        print("Booking Time     :", self.booking_time)

        if self.cancellation_time:
            print("Cancellation Time:", self.cancellation_time)

        print("-------------------------------")



# TRAIN CLASS


class Train:
    def __init__(self, train_no, train_name):

        self.train_no = train_no
        self.train_name = train_name

        # Dictionary containing seat numbers and status
        self.seats = {
            "SL": {},
            "3A": {},
            "2A": {},
            "FC": {}
        }

        # Creating seats
        for i in range(1, 11):
            self.seats["SL"][i] = "Available"

        for i in range(1, 6):
            self.seats["3A"][i] = "Available"

        for i in range(1, 4):
            self.seats["2A"][i] = "Available"

        for i in range(1, 3):
            self.seats["FC"][i] = "Available"

    def display_seats(self, travel_class):

        print("\nSeat Availability -", travel_class)
        print("--------------------------------")

        for seat, status in self.seats[travel_class].items():
            print("Seat", seat, ":", status)



# RESERVATION SYSTEM CLASS


class ReservationSystem:

    def __init__(self):

        self.passengers = []
        self.confirmed_tickets = []
        self.cancelled_tickets = []

        # FIFO waiting list
        self.waiting_list = []

        self.data_file = "railway_data.json"

        self.train = Train("12601", "Chennai Express")

        self.load_data()

    
    # GENERATE UNIQUE PNR
    
    def generate_pnr(self):

        while True:

            pnr = str(random.randint(1000000000, 9999999999))

            found = False

            for ticket in self.confirmed_tickets:
                if ticket.pnr == pnr:
                    found = True

            for ticket in self.cancelled_tickets:
                if ticket.pnr == pnr:
                    found = True

            if not found:
                return pnr

   
    # CHECK DUPLICATE BOOKING
    

    def duplicate_booking(self, name):

        for ticket in self.confirmed_tickets:

            if (ticket.passenger.name.lower() == name.lower()
                    and ticket.status == "Confirmed"):

                return True

        for item in self.waiting_list:

            if item["passenger"].name.lower() == name.lower():

                return True

        return False

    
    # CALCULATE FARE
    
    def calculate_fare(self, travel_class, passenger_type):

        fares = {
            "SL": 300,
            "3A": 700,
            "2A": 1000,
            "FC": 1500
        }

        fare = fares[travel_class]

        if passenger_type.lower() == "child":
            fare = fare * 0.5

        elif passenger_type.lower() == "senior":
            fare = fare * 0.7

        return fare

    
    # BOOK TICKET
    

    def book_ticket(self):

        print("\n========== BOOK TICKET ==========")

        name = input("Enter passenger name: ")

        if self.duplicate_booking(name):

            print("\nPassenger already has an active booking!")
            return

        try:
            age = int(input("Enter age: "))

            if age <= 0:
                print("Invalid age!")
                return

        except ValueError:
            print("Age must be a number!")
            return

        print("\nPassenger Type")
        print("1. Adult")
        print("2. Child")
        print("3. Senior")

        choice = input("Enter choice: ")

        if choice == "1":
            passenger_type = "Adult"

        elif choice == "2":
            passenger_type = "Child"

        elif choice == "3":
            passenger_type = "Senior"

        else:
            print("Invalid passenger type!")
            return

        print("\nTravel Classes")
        print("SL - Sleeper")
        print("3A - AC 3-Tier")
        print("2A - AC 2-Tier")
        print("FC - First Class")

        travel_class = input("Enter class: ").upper()

        if travel_class not in self.train.seats:

            print("Invalid travel class!")
            return

        # Find first available seat
        available_seat = None

        for seat, status in self.train.seats[travel_class].items():

            if status == "Available":
                available_seat = seat
                break

        passenger_id = len(self.passengers) + 1

        passenger = Passenger(
            passenger_id,
            name,
            age,
            passenger_type
        )

        self.passengers.append(passenger)

        fare = self.calculate_fare(
            travel_class,
            passenger_type
        )

       
        # CONFIRMED BOOKING
        

        if available_seat is not None:

            pnr = self.generate_pnr()

            self.train.seats[travel_class][available_seat] = "Occupied"

            ticket = Ticket(
                pnr,
                passenger,
                self.train.train_no,
                travel_class,
                available_seat,
                fare,
                "Confirmed"
            )

            self.confirmed_tickets.append(ticket)

            self.save_data()

            print("\n***** TICKET CONFIRMED *****")

            ticket.display()

        # WAITING LIST
        
        else:

            waiting_number = len(self.waiting_list) + 1

            pnr = self.generate_pnr()

            waiting_data = {
                "pnr": pnr,
                "passenger": passenger,
                "train_no": self.train.train_no,
                "travel_class": travel_class,
                "fare": fare,
                "waiting_number": waiting_number,
                "booking_time":
                    datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            }

            self.waiting_list.append(waiting_data)

            self.save_data()

            print("\nAll seats are occupied.")
            print("Passenger added to waiting list.")

            print("PNR :", pnr)
            print("Waiting Number :", waiting_number)

    
    # CANCEL TICKET
    

    def cancel_ticket(self):

        print("\n========== CANCEL TICKET ==========")

        pnr = input("Enter PNR: ")

        ticket_found = None

        for ticket in self.confirmed_tickets:

            if ticket.pnr == pnr:
                ticket_found = ticket
                break

        if ticket_found is None:

            print("Invalid PNR or ticket already cancelled!")
            return

        # Calculate cancellation charges
        booking_time = datetime.strptime(
            ticket_found.booking_time,
            "%Y-%m-%d %H:%M:%S"
        )

        current_time = datetime.now()

        hours = (
            current_time - booking_time
        ).total_seconds() / 3600

        if hours < 2:
            charge_percent = 10

        elif hours < 6:
            charge_percent = 25

        elif hours < 24:
            charge_percent = 50

        else:
            charge_percent = 75

        cancellation_charge = (
            ticket_found.fare * charge_percent / 100
        )

        refund = ticket_found.fare - cancellation_charge

        # Free the seat
        self.train.seats[
            ticket_found.travel_class
        ][ticket_found.seat_no] = "Available"

        ticket_found.status = "Cancelled"

        ticket_found.cancellation_time = (
            datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        )

        self.confirmed_tickets.remove(ticket_found)

        self.cancelled_tickets.append(ticket_found)

        print("\nTicket cancelled successfully!")

        print("Original Fare       :", ticket_found.fare)
        print("Cancellation Charge :", cancellation_charge)
        print("Refund Amount       :", refund)

        # Promote waiting passenger
        self.promote_waiting(ticket_found.travel_class)

        self.save_data()

   
    # PROMOTE WAITING LIST
   

    def promote_waiting(self, travel_class):

        eligible = None

        for item in self.waiting_list:

            if item["travel_class"] == travel_class:

                eligible = item
                break

        if eligible is None:
            return

        # Remove from waiting list
        self.waiting_list.remove(eligible)

        passenger = eligible["passenger"]

        # Find first available seat
        seat_no = None

        for seat, status in self.train.seats[
            travel_class
        ].items():

            if status == "Available":

                seat_no = seat
                break

        if seat_no is None:
            return

        self.train.seats[
            travel_class
        ][seat_no] = "Occupied"

        ticket = Ticket(
            eligible["pnr"],
            passenger,
            self.train.train_no,
            travel_class,
            seat_no,
            eligible["fare"],
            "Confirmed"
        )

        self.confirmed_tickets.append(ticket)

        print("\n***** WAITING LIST PROMOTION *****")

        print(
            passenger.name,
            "has been promoted to confirmed booking."
        )

        print("PNR :", ticket.pnr)
        print("Seat :", ticket.seat_no)

  
    # SEARCH BY PNR
   

    def search_pnr(self):

        print("\n========== PNR SEARCH ==========")

        pnr = input("Enter PNR: ")

        for ticket in self.confirmed_tickets:

            if ticket.pnr == pnr:

                ticket.display()
                return

        for ticket in self.cancelled_tickets:

            if ticket.pnr == pnr:

                ticket.display()
                return

        for item in self.waiting_list:

            if item["pnr"] == pnr:

                print("\nPNR :", item["pnr"])
                print(
                    "Passenger :",
                    item["passenger"].name
                )
                print(
                    "Class :",
                    item["travel_class"]
                )
                print(
                    "Waiting Number :",
                    item["waiting_number"]
                )

                return

        print("Invalid PNR!")

  
    # SEARCH PASSENGER
    

    def search_passenger(self):

        print("\n========== PASSENGER SEARCH ==========")

        name = input("Enter passenger name: ")

        found = False

        for ticket in self.confirmed_tickets:

            if ticket.passenger.name.lower() == name.lower():

                ticket.display()
                found = True

        for item in self.waiting_list:

            if item["passenger"].name.lower() == name.lower():

                print("\nWaiting List Passenger")
                print("Name :", item["passenger"].name)
                print("PNR :", item["pnr"])
                print("Class :", item["travel_class"])
                print(
                    "Waiting Number :",
                    item["waiting_number"]
                )

                found = True

        if not found:
            print("Passenger not found.")

   
    # SEAT AVAILABILITY
  

    def seat_availability(self):

        print("\n========== SEAT AVAILABILITY ==========")

        print("SL - Sleeper")
        print("3A - AC 3-Tier")
        print("2A - AC 2-Tier")
        print("FC - First Class")

        travel_class = input(
            "Enter travel class: "
        ).upper()

        if travel_class not in self.train.seats:

            print("Invalid class!")
            return

        self.train.display_seats(travel_class)


    # DISPLAY WAITING LIST  

    def display_waiting_list(self):

        print("\n========== WAITING LIST ==========")

        if len(self.waiting_list) == 0:

            print("Waiting list is empty.")
            return

        for item in self.waiting_list:

            print("\nWaiting Number :", item["waiting_number"])
            print("PNR            :", item["pnr"])
            print(
                "Passenger      :",
                item["passenger"].name
            )
            print(
                "Class          :",
                item["travel_class"]
            )
            print(
                "Fare           :",
                item["fare"]
            )

   
    # DISPLAY ALL CONFIRMED TICKETS
    # 

    def display_confirmed(self):

        print("\n========== CONFIRMED TICKETS ==========")

        if len(self.confirmed_tickets) == 0:

            print("No confirmed tickets.")
            return

        for ticket in self.confirmed_tickets:

            ticket.display()

    
    # SAVE DATA
    

    def save_data(self):

        data = {
            "confirmed": [],
            "cancelled": [],
            "waiting": []
        }

        for ticket in self.confirmed_tickets:

            data["confirmed"].append(
                self.ticket_to_dict(ticket)
            )

        for ticket in self.cancelled_tickets:

            data["cancelled"].append(
                self.ticket_to_dict(ticket)
            )

        for item in self.waiting_list:

            data["waiting"].append({
                "pnr": item["pnr"],
                "passenger": {
                    "passenger_id":
                        item["passenger"].passenger_id,
                    "name":
                        item["passenger"].name,
                    "age":
                        item["passenger"].age,
                    "passenger_type":
                        item["passenger"].passenger_type
                },
                "train_no":
                    item["train_no"],
                "travel_class":
                    item["travel_class"],
                "fare":
                    item["fare"],
                "waiting_number":
                    item["waiting_number"],
                "booking_time":
                    item["booking_time"]
            })

        with open(self.data_file, "w") as file:

            json.dump(
                data,
                file,
                indent=4
            )

    #
    # CONVERT TICKET TO DICTIONARY
    # 

    def ticket_to_dict(self, ticket):

        return {
            "pnr": ticket.pnr,
            "passenger": {
                "passenger_id":
                    ticket.passenger.passenger_id,
                "name":
                    ticket.passenger.name,
                "age":
                    ticket.passenger.age,
                "passenger_type":
                    ticket.passenger.passenger_type
            },
            "train_no": ticket.train_no,
            "travel_class": ticket.travel_class,
            "seat_no": ticket.seat_no,
            "fare": ticket.fare,
            "status": ticket.status,
            "booking_time": ticket.booking_time,
            "cancellation_time":
                ticket.cancellation_time
        }

    # 
    # LOAD DATA
    # 

    def load_data(self):

        if not os.path.exists(self.data_file):
            return

        try:

            with open(self.data_file, "r") as file:

                data = json.load(file)

            # Load confirmed tickets
            for item in data.get("confirmed", []):

                p = item["passenger"]

                passenger = Passenger(
                    p["passenger_id"],
                    p["name"],
                    p["age"],
                    p["passenger_type"]
                )

                self.passengers.append(passenger)

                ticket = Ticket(
                    item["pnr"],
                    passenger,
                    item["train_no"],
                    item["travel_class"],
                    item["seat_no"],
                    item["fare"],
                    item["status"]
                )

                ticket.booking_time = item["booking_time"]

                self.confirmed_tickets.append(ticket)

                # Mark seat occupied
                if item["seat_no"] in self.train.seats[
                    item["travel_class"]
                ]:

                    self.train.seats[
                        item["travel_class"]
                    ][item["seat_no"]] = "Occupied"

            # Load cancelled tickets
            for item in data.get("cancelled", []):

                p = item["passenger"]

                passenger = Passenger(
                    p["passenger_id"],
                    p["name"],
                    p["age"],
                    p["passenger_type"]
                )

                ticket = Ticket(
                    item["pnr"],
                    passenger,
                    item["train_no"],
                    item["travel_class"],
                    item["seat_no"],
                    item["fare"],
                    item["status"]
                )

                ticket.booking_time = item["booking_time"]

                ticket.cancellation_time = item[
                    "cancellation_time"
                ]

                self.cancelled_tickets.append(ticket)

            # Load waiting list
            for item in data.get("waiting", []):

                p = item["passenger"]

                passenger = Passenger(
                    p["passenger_id"],
                    p["name"],
                    p["age"],
                    p["passenger_type"]
                )

                self.passengers.append(passenger)

                waiting_data = {
                    "pnr": item["pnr"],
                    "passenger": passenger,
                    "train_no": item["train_no"],
                    "travel_class":
                        item["travel_class"],
                    "fare": item["fare"],
                    "waiting_number":
                        item["waiting_number"],
                    "booking_time":
                        item["booking_time"]
                }

                self.waiting_list.append(waiting_data)

        except Exception as e:

            print("Error loading data:", e)



# MAIN MENU


def main():

    system = ReservationSystem()

    while True:

        print("\n")
        print("========================================")
        print("     RAILWAY RESERVATION SYSTEM")
        print("========================================")
        print("1. Book Ticket")
        print("2. Cancel Ticket")
        print("3. Search by PNR")
        print("4. Search Passenger")
        print("5. Seat Availability")
        print("6. Display Waiting List")
        print("7. Display Confirmed Tickets")
        print("8. Exit")
        print("========================================")

        choice = input("Enter your choice: ")

        if choice == "1":

            system.book_ticket()

        elif choice == "2":

            system.cancel_ticket()

        elif choice == "3":

            system.search_pnr()

        elif choice == "4":

            system.search_passenger()

        elif choice == "5":

            system.seat_availability()

        elif choice == "6":

            system.display_waiting_list()

        elif choice == "7":

            system.display_confirmed()

        elif choice == "8":

            print("\nThank you for using Railway Reservation System!")

            system.save_data()

            break

        else:

            print("\nInvalid menu choice!")
            print("Please enter a number from 1 to 8.")


# PROGRAM START

#2 project 
#Smart Hotel Reservation & Room Allocation System

import random
import time
import json
import os


# ==========================================================
# CUSTOMER CLASS
# ==========================================================

class Customer:

    def __init__(self, customer_id, name, age, phone):
        self.customer_id = customer_id
        self.name = name
        self.age = age
        self.phone = phone

    def display(self):
        print("\n-----------------------------")
        print("Customer ID :", self.customer_id)
        print("Name        :", self.name)
        print("Age         :", self.age)
        print("Phone       :", self.phone)
        print("-----------------------------")


# ==========================================================
# ROOM CLASS
# ==========================================================

class Room:

    def __init__(self, room_number, room_type, price):
        self.room_number = room_number
        self.room_type = room_type
        self.price = price
        self.status = "Available"

    def display(self):
        print(
            "Room:", self.room_number,
            "| Type:", self.room_type,
            "| Price:", self.price,
            "| Status:", self.status
        )


# ==========================================================
# RESERVATION CLASS
# ==========================================================

class Reservation:

    def __init__(
        self,
        booking_id,
        customer,
        room,
        check_in_date,
        check_out_date,
        days
    ):

        self.booking_id = booking_id
        self.customer = customer
        self.room = room
        self.check_in_date = check_in_date
        self.check_out_date = check_out_date
        self.days = days

        self.status = "Reserved"

        self.reservation_time = time.ctime()
        self.checkin_time = None
        self.checkout_time = None

        self.bill = 0
        self.cancellation_charge = 0

    def display(self):

        print("\n================================")
        print("Booking ID      :", self.booking_id)
        print("Customer        :", self.customer.name)
        print("Phone           :", self.customer.phone)
        print("Room Number     :", self.room.room_number)
        print("Room Type       :", self.room.room_type)
        print("Check-in Date   :", self.check_in_date)
        print("Check-out Date  :", self.check_out_date)
        print("Number of Days  :", self.days)
        print("Status          :", self.status)
        print("Reservation Time:", self.reservation_time)

        if self.checkin_time:
            print("Check-in Time   :", self.checkin_time)

        if self.checkout_time:
            print("Check-out Time  :", self.checkout_time)

        if self.bill > 0:
            print("Bill            :", self.bill)

        print("================================")


# ==========================================================
# HOTEL CLASS
# ==========================================================

class Hotel:

    def __init__(self):

        self.customers = []

        # Dictionary using booking ID as key
        self.reservations = {}

        self.booking_history = []

        self.rooms = []

        self.data_file = "hotel_data.json"

        self.create_rooms()

        self.load_data()

    # ------------------------------------------------------
    # CREATE ROOMS
    # ------------------------------------------------------

    def create_rooms(self):

        # Single rooms
        for i in range(101, 106):
            self.rooms.append(
                Room(i, "Single", 1500)
            )

        # Double rooms
        for i in range(201, 206):
            self.rooms.append(
                Room(i, "Double", 2500)
            )

        # Deluxe rooms
        for i in range(301, 304):
            self.rooms.append(
                Room(i, "Deluxe", 4000)
            )

        # Suite rooms
        for i in range(401, 403):
            self.rooms.append(
                Room(i, "Suite", 6000)
            )

    # ------------------------------------------------------
    # GENERATE UNIQUE BOOKING ID
    # ------------------------------------------------------

    def generate_booking_id(self):

        while True:

            booking_id = "BK" + str(
                random.randint(10000, 99999)
            )

            if booking_id not in self.reservations:
                return booking_id

    # ------------------------------------------------------
    # ADD CUSTOMER
    # ------------------------------------------------------

    def add_customer(self):

        print("\n========== ADD CUSTOMER ==========")

        name = input("Enter customer name: ").strip()

        if name == "":
            print("Name cannot be empty!")
            return

        try:
            age = int(input("Enter age: "))

            if age <= 0:
                print("Invalid age!")
                return

        except ValueError:
            print("Age must be a number!")
            return

        phone = input("Enter phone number: ").strip()

        if not phone.isdigit():
            print("Phone number must contain only digits!")
            return

        customer_id = len(self.customers) + 1

        customer = Customer(
            customer_id,
            name,
            age,
            phone
        )

        self.customers.append(customer)

        self.save_data()

        print("\nCustomer added successfully!")
        print("Customer ID:", customer_id)

    # ------------------------------------------------------
    # FIND CUSTOMER
    # ------------------------------------------------------

    def find_customer(self, customer_id):

        for customer in self.customers:

            if customer.customer_id == customer_id:
                return customer

        return None

    # ------------------------------------------------------
    # FIND AVAILABLE ROOM
    # ------------------------------------------------------

    def find_available_room(self, room_type):

        for room in self.rooms:

            if (
                room.room_type == room_type
                and room.status == "Available"
            ):
                return room

        return None

    # ------------------------------------------------------
    # CALCULATE BILL
    # ------------------------------------------------------

    def calculate_bill(self, room, days):

        basic_amount = room.price * days

        # Weekend charge
        weekend_charge = 0

        if days >= 2:
            weekend_charge = basic_amount * 0.10

        total = basic_amount + weekend_charge

        # Discount for long stay
        discount = 0

        if days > 5:
            discount = total * 0.15

        total = total - discount

        return total

    # ------------------------------------------------------
    # BOOK ROOM
    # ------------------------------------------------------

    def book_room(self):

        print("\n========== BOOK ROOM ==========")

        if len(self.customers) == 0:

            print("Please add a customer first!")
            return

        try:

            customer_id = int(
                input("Enter customer ID: ")
            )

        except ValueError:

            print("Invalid customer ID!")
            return

        customer = self.find_customer(customer_id)

        if customer is None:

            print("Customer not found!")
            return

        print("\nRoom Types")

        print("1. Single")
        print("2. Double")
        print("3. Deluxe")
        print("4. Suite")

        choice = input("Enter choice: ")

        room_types = {
            "1": "Single",
            "2": "Double",
            "3": "Deluxe",
            "4": "Suite"
        }

        if choice not in room_types:

            print("Invalid room type!")
            return

        room_type = room_types[choice]

        try:

            days = int(
                input("Enter number of days: ")
            )

            if days <= 0:
                print("Days must be greater than zero!")
                return

        except ValueError:

            print("Invalid number of days!")
            return

        # Find available room
        room = self.find_available_room(room_type)

        if room is None:

            print("\nNo", room_type, "rooms available!")
            return

        check_in = input(
            "Enter check-in date (DD-MM-YYYY): "
        )

        check_out = input(
            "Enter check-out date (DD-MM-YYYY): "
        )

        # Generate booking ID
        booking_id = self.generate_booking_id()

        # Calculate bill
        bill = self.calculate_bill(
            room,
            days
        )

        # Create reservation
        reservation = Reservation(
            booking_id,
            customer,
            room,
            check_in,
            check_out,
            days
        )

        reservation.bill = bill

        # Change room status
        room.status = "Reserved"

        # Store reservation in dictionary
        self.reservations[
            booking_id
        ] = reservation

        self.save_data()

        print("\n***** ROOM BOOKED SUCCESSFULLY *****")

        reservation.display()

    # ------------------------------------------------------
    # CANCEL BOOKING
    # ------------------------------------------------------

    def cancel_booking(self):

        print("\n========== CANCEL BOOKING ==========")

        booking_id = input(
            "Enter booking ID: "
        ).upper()

        if booking_id not in self.reservations:

            print("Invalid booking ID!")
            return

        reservation = self.reservations[
            booking_id
        ]

        if reservation.status in [
            "Cancelled",
            "Checked-Out"
        ]:

            print("Booking cannot be cancelled!")
            return

        # Cancellation charge = 10%
        charge = reservation.bill * 0.10

        refund = reservation.bill - charge

        reservation.cancellation_charge = charge
        reservation.status = "Cancelled"

        # Make room available
        reservation.room.status = "Available"

        # Add to history
        self.booking_history.append(
            reservation
        )

        self.save_data()

        print("\nBooking cancelled successfully!")

        print("Booking Amount     :", reservation.bill)
        print("Cancellation Charge:", charge)
        print("Refund Amount      :", refund)

    # ------------------------------------------------------
    # CHECK IN
    # ------------------------------------------------------

    def check_in(self):

        print("\n========== CHECK IN ==========")

        booking_id = input(
            "Enter booking ID: "
        ).upper()

        if booking_id not in self.reservations:

            print("Invalid booking ID!")
            return

        reservation = self.reservations[
            booking_id
        ]

        if reservation.status != "Reserved":

            print(
                "Only reserved bookings can be checked in!"
            )

            return

        reservation.status = "Checked-In"

        reservation.checkin_time = time.ctime()

        reservation.room.status = "Checked-In"

        self.save_data()

        print("\nCheck-in successful!")
        print(
            "Check-in Time:",
            reservation.checkin_time
        )

    # ------------------------------------------------------
    # CHECK OUT
    # ------------------------------------------------------

    def check_out(self):

        print("\n========== CHECK OUT ==========")

        booking_id = input(
            "Enter booking ID: "
        ).upper()

        if booking_id not in self.reservations:

            print("Invalid booking ID!")
            return

        reservation = self.reservations[
            booking_id
        ]

        if reservation.status != "Checked-In":

            print(
                "Customer has not checked in!"
            )

            return

        # Late checkout
        print("\nAllowed checkout time: 12:00 PM")

        late = input(
            "Is the customer checking out late? (yes/no): "
        ).lower()

        late_charge = 0

        if late == "yes":

            late_charge = 500

            print(
                "Late checkout charge:",
                late_charge
            )

        elif late != "no":

            print("Invalid choice!")
            return

        reservation.bill += late_charge

        reservation.status = "Checked-Out"

        reservation.checkout_time = time.ctime()

        reservation.room.status = "Available"

        # Add to booking history
        self.booking_history.append(
            reservation
        )

        self.save_data()

        print("\n***** CHECK OUT SUCCESSFUL *****")

        print(
            "Final Bill:",
            reservation.bill
        )

        print(
            "Checkout Time:",
            reservation.checkout_time
        )

    # ------------------------------------------------------
    # SEARCH BOOKING
    # ------------------------------------------------------

    def search_booking(self):

        print("\n========== SEARCH BOOKING ==========")

        booking_id = input(
            "Enter booking ID: "
        ).upper()

        if booking_id in self.reservations:

            self.reservations[
                booking_id
            ].display()

            return

        # Search history
        for reservation in self.booking_history:

            if reservation.booking_id == booking_id:

                reservation.display()
                return

        print("Booking not found!")

    # ------------------------------------------------------
    # DISPLAY ROOMS
    # ------------------------------------------------------

    def display_rooms(self):

        print("\n========== ROOM DETAILS ==========")

        for room in self.rooms:
            room.display()

    # ------------------------------------------------------
    # DISPLAY BOOKING HISTORY
    # ------------------------------------------------------

    def display_history(self):

        print("\n========== BOOKING HISTORY ==========")

        if len(self.booking_history) == 0:

            print("No booking history available.")
            return

        for reservation in self.booking_history:

            reservation.display()

    # ------------------------------------------------------
    # SAVE DATA
    # ------------------------------------------------------

    def save_data(self):

        data = {
            "customers": [],
            "reservations": {}
        }

        # Save customers
        for customer in self.customers:

            data["customers"].append({

                "customer_id":
                    customer.customer_id,

                "name":
                    customer.name,

                "age":
                    customer.age,

                "phone":
                    customer.phone
            })

        # Save reservations
        for booking_id, reservation in self.reservations.items():

            data["reservations"][booking_id] = {

                "booking_id":
                    reservation.booking_id,

                "customer_id":
                    reservation.customer.customer_id,

                "room_number":
                    reservation.room.room_number,

                "check_in_date":
                    reservation.check_in_date,

                "check_out_date":
                    reservation.check_out_date,

                "days":
                    reservation.days,

                "status":
                    reservation.status,

                "bill":
                    reservation.bill,

                "reservation_time":
                    reservation.reservation_time,

                "checkin_time":
                    reservation.checkin_time,

                "checkout_time":
                    reservation.checkout_time
            }

        with open(
            self.data_file,
            "w"
        ) as file:

            json.dump(
                data,
                file,
                indent=4
            )

    # ------------------------------------------------------
    # LOAD DATA
    # ------------------------------------------------------

    def load_data(self):

        if not os.path.exists(
            self.data_file
        ):
            return

        try:

            with open(
                self.data_file,
                "r"
            ) as file:

                data = json.load(file)

            # Load customers
            for item in data.get(
                "customers",
                []
            ):

                customer = Customer(
                    item["customer_id"],
                    item["name"],
                    item["age"],
                    item["phone"]
                )

                self.customers.append(
                    customer
                )

            # Load reservations
            for booking_id, item in data.get(
                "reservations",
                {}
            ).items():

                customer = self.find_customer(
                    item["customer_id"]
                )

                room = None

                for r in self.rooms:

                    if (
                        r.room_number
                        == item["room_number"]
                    ):

                        room = r
                        break

                if customer is None or room is None:
                    continue

                reservation = Reservation(
                    item["booking_id"],
                    customer,
                    room,
                    item["check_in_date"],
                    item["check_out_date"],
                    item["days"]
                )

                reservation.status = item["status"]

                reservation.bill = item["bill"]

                reservation.reservation_time = item[
                    "reservation_time"
                ]

                reservation.checkin_time = item[
                    "checkin_time"
                ]

                reservation.checkout_time = item[
                    "checkout_time"
                ]

                self.reservations[
                    booking_id
                ] = reservation

                # Restore room status
                if reservation.status == "Reserved":

                    room.status = "Reserved"

                elif reservation.status == "Checked-In":

                    room.status = "Checked-In"

        except Exception as e:

            print(
                "Error loading data:",
                e
            )


# ==========================================================
# MAIN PROGRAM
# ==========================================================

hotel = Hotel()


while True:

    print("\n")
    print("==========================================")
    print("       SMART HOTEL RESERVATION SYSTEM")
    print("==========================================")

    print("1. Add Customer")
    print("2. Book Room")
    print("3. Cancel Booking")
    print("4. Check In")
    print("5. Check Out")
    print("6. Search Booking")
    print("7. View Rooms")
    print("8. View Booking History")
    print("9. Save Data")
    print("10. Exit")

    print("==========================================")

    choice = input(
        "Enter your choice: "
    )

    if choice == "1":

        hotel.add_customer()

    elif choice == "2":

        hotel.book_room()

    elif choice == "3":

        hotel.cancel_booking()

    elif choice == "4":

        hotel.check_in()

    elif choice == "5":

        hotel.check_out()

    elif choice == "6":

        hotel.search_booking()

    elif choice == "7":

        hotel.display_rooms()

    elif choice == "8":

        hotel.display_history()

    elif choice == "9":

        hotel.save_data()

        print("Data saved successfully!")

    elif choice == "10":

        hotel.save_data()

        print("\nThank you for using the Hotel System!")

        break

    else:

        print(
            "Invalid menu choice! "
            "Please enter a valid option."
        )

#3 project 
#Multiplayer Number Guessing Championship
import random
import time
import json
import os


# ==========================================================
# PLAYER CLASS
# ==========================================================

class Player:

    def __init__(self, name):
        self.name = name
        self.score = 0
        self.attempts = 0
        self.games_played = 0
        self.games_won = 0
        self.best_score = 0

    def display(self):

        print("\n-----------------------------")
        print("Player Name   :", self.name)
        print("Total Score   :", self.score)
        print("Games Played  :", self.games_played)
        print("Games Won     :", self.games_won)
        print("Total Attempts:", self.attempts)
        print("Best Score    :", self.best_score)
        print("-----------------------------")


# ==========================================================
# CHAMPIONSHIP CLASS
# ==========================================================

class NumberGuessingChampionship:

    def __init__(self):

        # Dictionary to store player statistics
        self.players = {}

        self.leaderboard_file = "leaderboard.json"

        self.load_leaderboard()

    # ------------------------------------------------------
    # LOAD LEADERBOARD
    # ------------------------------------------------------

    def load_leaderboard(self):

        if not os.path.exists(
            self.leaderboard_file
        ):
            return

        try:

            with open(
                self.leaderboard_file,
                "r"
            ) as file:

                data = json.load(file)

            for name, details in data.items():

                player = Player(name)

                player.score = details["score"]
                player.attempts = details["attempts"]
                player.games_played = details[
                    "games_played"
                ]
                player.games_won = details[
                    "games_won"
                ]
                player.best_score = details[
                    "best_score"
                ]

                self.players[name] = player

        except Exception as e:

            print(
                "Error loading leaderboard:",
                e
            )

    # ------------------------------------------------------
    # SAVE LEADERBOARD
    # ------------------------------------------------------

    def save_leaderboard(self):

        data = {}

        for name, player in self.players.items():

            data[name] = {

                "score": player.score,

                "attempts": player.attempts,

                "games_played":
                    player.games_played,

                "games_won":
                    player.games_won,

                "best_score":
                    player.best_score
            }

        with open(
            self.leaderboard_file,
            "w"
        ) as file:

            json.dump(
                data,
                file,
                indent=4
            )

    # ------------------------------------------------------
    # DISPLAY RULES
    # ------------------------------------------------------

    def display_rules(self):

        print("\n======================================")
        print("        NUMBER GUESSING RULES")
        print("======================================")

        print("1. Number of players: 2 to 5")
        print("2. Player names must be unique.")
        print("3. Players get a limited number of attempts.")
        print("4. Every wrong guess reduces points.")
        print("5. Too High / Too Low hints are provided.")
        print("6. Very Close hint is given for nearby guesses.")
        print("7. Correct guesses receive points.")
        print("8. Fast correct guesses receive a time bonus.")
        print("9. Players who use all attempts get 0 points.")
        print("10. Player order is randomized.")
        print("11. Scores are saved after every game.")

        print("======================================")

    # ------------------------------------------------------
    # GET DIFFICULTY
    # ------------------------------------------------------

    def choose_difficulty(self):

        print("\n========== DIFFICULTY ==========")

        print("1. Easy   (1 - 50)")
        print("2. Medium (1 - 100)")
        print("3. Hard   (1 - 500)")

        choice = input(
            "Enter difficulty: "
        )

        if choice == "1":

            return "Easy", 50

        elif choice == "2":

            return "Medium", 100

        elif choice == "3":

            return "Hard", 500

        else:

            print("Invalid difficulty choice!")
            return None, None

    # ------------------------------------------------------
    # CREATE PLAYERS
    # ------------------------------------------------------

    def create_players(self):

        print("\n========== PLAYERS ==========")

        while True:

            try:

                count = int(
                    input(
                        "Enter number of players (2-5): "
                    )
                )

                if 2 <= count <= 5:
                    break

                print(
                    "Number of players must be between 2 and 5."
                )

            except ValueError:

                print(
                    "Please enter a valid number."
                )

        players = []

        for i in range(count):

            while True:

                name = input(
                    f"Enter name of Player {i + 1}: "
                ).strip()

                if name == "":
                    print("Name cannot be empty!")
                    continue

                # Prevent duplicate names
                duplicate = False

                for player in players:

                    if player.name.lower() == name.lower():

                        duplicate = True
                        break

                if duplicate:

                    print(
                        "This player name already exists!"
                    )

                    continue

                # Use existing leaderboard player
                if name in self.players:

                    player = self.players[name]

                else:

                    player = Player(name)

                    self.players[name] = player

                players.append(player)

                break

        return players

    # ------------------------------------------------------
    # START NEW GAME
    # ------------------------------------------------------

    def start_game(self):

        print("\n======================================")
        print("     MULTIPLAYER GUESSING GAME")
        print("======================================")

        players = self.create_players()

        difficulty, maximum = self.choose_difficulty()

        if difficulty is None:
            return

        # Generate secret number
        secret_number = random.randint(
            1,
            maximum
        )

        # Randomize player order
        random.shuffle(players)

        print("\nDifficulty:", difficulty)
        print(
            "Secret number is between 1 and",
            maximum
        )

        print("\nPlayer order:")

        for i, player in enumerate(players):

            print(
                i + 1,
                ".",
                player.name
            )

        # Number of attempts
        attempts_allowed = 5

        # Starting points
        starting_points = 100

        # Store game results
        game_scores = {}

        # --------------------------------------------------
        # EACH PLAYER'S TURN
        # --------------------------------------------------

        for player in players:

            print("\n================================")
            print(
                "Turn:",
                player.name
            )
            print("================================")

            print(
                "You have",
                attempts_allowed,
                "attempts."
            )

            points = starting_points
            correct = False

            player_attempts = 0

            # Start timer
            start_time = time.time()

            for attempt in range(
                1,
                attempts_allowed + 1
            ):

                while True:

                    guess_input = input(
                        f"Attempt {attempt}: "
                    )

                    try:

                        guess = int(
                            guess_input
                        )

                        if (
                            guess < 1
                            or guess > maximum
                        ):

                            print(
                                "Guess must be between 1 and",
                                maximum
                            )

                            continue

                        break

                    except ValueError:

                        print(
                            "Invalid input! "
                            "Enter a number."
                        )

                player_attempts += 1
                player.attempts += 1

                # Correct answer
                if guess == secret_number:

                    end_time = time.time()

                    time_taken = (
                        end_time - start_time
                    )

                    # Time bonus
                    if time_taken <= 5:

                        time_bonus = 50

                    elif time_taken <= 10:

                        time_bonus = 30

                    elif time_taken <= 20:

                        time_bonus = 15

                    else:

                        time_bonus = 5

                    points += time_bonus

                    # Prevent negative score
                    if points < 0:
                        points = 0

                    print("\n***** CORRECT! *****")

                    print(
                        "Time Taken:",
                        round(time_taken, 2),
                        "seconds"
                    )

                    print(
                        "Time Bonus:",
                        time_bonus
                    )

                    print(
                        "Points:",
                        points
                    )

                    correct = True

                    player.score += points

                    player.games_won += 1

                    player.games_played += 1

                    if points > player.best_score:

                        player.best_score = points

                    game_scores[player.name] = points

                    break

                # Wrong answer
                else:

                    # Reduce points
                    points -= 20

                    if points < 0:
                        points = 0

                    difference = abs(
                        secret_number - guess
                    )

                    if guess > secret_number:

                        print("Too High!")

                    else:

                        print("Too Low!")

                    # Very close hint
                    if difference <= 5:

                        print("Very Close!")

                    print(
                        "Points remaining:",
                        points
                    )

            # Player failed
            if not correct:

                print("\nNo attempts remaining!")

                print(
                    "The secret number was:",
                    secret_number
                )

                print(
                    "You receive 0 points."
                )

                game_scores[player.name] = 0

                player.games_played += 1

        # --------------------------------------------------
        # GAME RESULT
        # --------------------------------------------------

        print("\n======================================")
        print("           GAME RESULTS")
        print("======================================")

        for name, points in game_scores.items():

            print(
                name,
                "=>",
                points,
                "points"
            )

        # Determine winner
        winner = max(
            game_scores,
            key=game_scores.get
        )

        highest_score = game_scores[winner]

        print("\n***** WINNER *****")

        print(
            winner,
            "with",
            highest_score,
            "points!"
        )

        # Save after completed game
        self.save_leaderboard()

        print(
            "\nLeaderboard saved successfully!"
        )

    # ------------------------------------------------------
    # SHOW LEADERBOARD
    # ------------------------------------------------------

    def show_leaderboard(self):

        print("\n======================================")
        print("             LEADERBOARD")
        print("======================================")

        if len(self.players) == 0:

            print("No player statistics available.")
            return

        # Sort players according to score
        sorted_players = sorted(
            self.players.values(),
            key=lambda p: p.score,
            reverse=True
        )

        for position, player in enumerate(
            sorted_players,
            start=1
        ):

            print(
                position,
                ".",
                player.name,
                "-",
                player.score,
                "points"
            )

    # ------------------------------------------------------
    # SEARCH PLAYER STATISTICS
    # ------------------------------------------------------

    def search_player(self):

        print("\n========== SEARCH PLAYER ==========")

        name = input(
            "Enter player name: "
        ).strip()

        player = None

        for player_name, p in self.players.items():

            if player_name.lower() == name.lower():

                player = p
                break

        if player is None:

            print("Player not found!")
            return

        player.display()


# ==========================================================
# MAIN PROGRAM
# ==========================================================

game = NumberGuessingChampionship()


while True:

    print("\n")
    print("======================================")
    print(" MULTIPLAYER NUMBER GUESSING CHAMPIONSHIP")
    print("======================================")

    print("1. Start New Game")
    print("2. Display Rules")
    print("3. Show Leaderboard")
    print("4. Search Player Statistics")
    print("5. Exit")

    print("======================================")

    choice = input(
        "Enter your choice: "
    )

    if choice == "1":

        game.start_game()

    elif choice == "2":

        game.display_rules()

    elif choice == "3":

        game.show_leaderboard()

    elif choice == "4":

        game.search_player()

    elif choice == "5":

        game.save_leaderboard()

        print(
            "\nThank you for playing!"
        )

        break

    else:

        print(
            "Invalid menu choice! "
            "Please enter 1-5."
        )
        






if __name__ == "__main__":
    main()
