# base image
FROM python:3.13

# working directory
WORKDIR /app/

# copy requirements.txt
COPY requirements.txt ./

# install python dependencies
RUN pip3 install --no-cache-dir -r requirements.txt

# Copy all source code to the app/frontend
COPY app.py .

# set ENV var to default value to 10000
ENV PORT=10000

# expose port for the app
EXPOSE ${PORT}

# to run flask server
CMD ["python3", "app.py"]